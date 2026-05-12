# Course Embeddings Agent

Generates vector embeddings from Brightspace course content and stores them as Parquet files within the course's file system. Other agents can load these Parquet files to perform RAG (Retrieval-Augmented Generation) against course materials.

## How It Works

The agent follows a multi-stage pipeline: **crawl → extract → chunk → embed → write**. Each stage is described below.

### 1. Content Crawling

The agent uses the Brightspace Valence API to enumerate the course's module structure. Each module contains topics (individual documents). The crawler fetches the raw content of each topic and computes a SHA-256 content hash for change detection.

Supported file types: **PDF, DOCX, PPTX, HTML, TXT, MD**.

#### API Content (optional, enabled by default)

When "Include API content" is enabled, the crawler also fetches instructor-authored text from the Brightspace API that doesn't exist as downloadable files:

| Source | API Route | Text Fields |
|--------|-----------|-------------|
| Module descriptions | `GET /le/{ver}/{ou}/content/modules/{moduleId}` | `Description` |
| Topic descriptions | `GET /le/{ver}/{ou}/content/topics/{topicId}` | `Description` |
| Discussion prompts | `GET /le/{ver}/{ou}/discussions/forums/{forumId}` | Forum `Name`/`Description`, topic `Name`/`Description` |
| Assignment instructions | `GET /le/{ver}/{ou}/dropbox/folders/{folderId}` | `Name`, `CustomInstructions` |
| Quiz instructions | `GET /le/{ver}/{ou}/quizzes/{quizId}` | `Name`, `Instructions`, `Description` |
| News/announcements | `GET /le/{ver}/{ou}/news/` | `Title`, `Body` |

Activities (discussions, assignments, quizzes) are scoped to modules via the content TOC -- only activities linked into a module's content area are included. News items are course-wide and placed in a virtual "Course Announcements" module.

Module descriptions appear as virtual "Overview" topics. Topic descriptions for file-based topics are prepended to the file's extracted text as a blockquote.

### 2. Text Extraction

Text extraction converts raw document bytes into structured markdown. The extraction method depends on file type and the selected extraction mode.

#### Programmatic Extraction

| File Type | Method | Output |
|-----------|--------|--------|
| HTML | Turndown (HTML → Markdown) | Markdown with headings, lists, tables |
| DOCX | Mammoth (`convertToHtml`) → Turndown | Markdown with headings, lists, formatting |
| PPTX | XML parsing with `## Slide N` headings | Markdown with slide boundaries and paragraphs |
| PDF | `pdfjs-dist` text layer extraction | Plain text (limited structure) |
| TXT / MD | Used as-is | Raw text / Markdown |

#### Vision Extraction (PDF only, default)

For PDFs, the agent can use a vision model to transcribe each page from a rendered image. This produces far more accurate and structured markdown than programmatic PDF text extraction, especially for scanned documents, complex layouts, tables, and mathematical notation.

The vision pipeline:
1. Renders each PDF page to a PNG image at 2× scale
2. Uploads the image to S3 via a pre-signed URL
3. Sends the image to a vision-capable LLM with the `transcribe-for-embeddings` prompt
4. Concatenates per-page markdown into a single document

Vision extraction is only used for PDFs. All other file types use programmatic extraction regardless of the setting.

### 3. Transcript Caching

After extraction, the markdown transcript is saved to Brightspace course files at:

```
embeddings/transcripts/{slugified-topic-title}.md
```

On subsequent indexing runs, the agent checks the content hash:
- If the source document hasn't changed (same hash), the cached transcript is loaded instead of re-extracting
- If the source document has changed (different hash), the transcript is regenerated
- If there is no hash in the manifest (e.g., after deletion), the agent optimistically tries to load the cached transcript before falling back to re-extraction

This avoids redundant vision model calls, which are the most expensive part of the pipeline.

### 4. Chunking

Chunking splits the extracted markdown into segments suitable for embedding. Two modes are available.

#### Rule-Based Chunking

A recursive text splitter that tries separators in order of structural significance:
1. Heading boundaries
2. Paragraph breaks
3. Sentence boundaries
4. Line breaks
5. Character limit (hard fallback)

Chunks include configurable overlap (default: 100 tokens) to preserve context across boundaries.

#### LLM-Assisted Chunking (default)

A two-pass approach that produces semantically coherent chunks:

**Pass 1 — Boundary Detection** (`chunk-boundaries.md`)

The extracted text is normalized (hard-wrapped prose lines are collapsed into single paragraphs) and numbered (`LINE_NUMBER| text`). The numbered text is sent to an LLM, which returns a JSON array of boundary markers:

```json
[
  { "title": "Short descriptive title", "startLine": 1, "endLine": 45, "discard": false },
  { "title": "References", "startLine": 46, "endLine": 52, "discard": true }
]
```

The actual chunk text is extracted programmatically from the line ranges — the LLM only decides *where* to split, it does not rewrite or paraphrase the content. Sections marked `discard: true` (references, boilerplate, page numbers) are excluded from indexing.

For documents that exceed ~400k characters, the text is automatically split into overlapping windows, each processed independently, and the results are stitched together with deduplication.

**Pass 2 — Context Preambles** (`chunk-context.md`)

A single LLM call receives the full document and a preview of each chunk. For each chunk, it generates a 1-3 sentence preamble that resolves ambiguities a reader would face seeing the chunk in isolation:
- Pronouns ("this", "they") → named referents
- Acronyms → expanded definitions
- Cross-references ("as discussed above") → brief restatements
- Implicit context → document/chapter/topic identification

The preamble is based solely on the source document text, not on LLM-generated titles or external labels.

If either LLM pass fails, the agent falls back to rule-based chunking automatically.

### 5. Context Assembly

Each chunk produces two text fields: `chunk_text` (the raw source text) and `context_text` (the enriched text that gets embedded). The `context_text` is assembled with explicit section labels:

```
[CONTEXT TEXT]
Module: {module title} | Topic: {topic title} | {chunk title}

{preamble from Pass 2, if any}

[SOURCE TEXT]
{original chunk text}
```

For rule-based chunking, the `context_text` has the same structure but without a preamble:

```
[CONTEXT TEXT]
Module: {module title} | Topic: {topic title} |

[SOURCE TEXT]
{original chunk text}
```

The `[CONTEXT TEXT]` and `[SOURCE TEXT]` labels allow downstream RAG consumers to reliably distinguish added context from original source material.

### 6. Embedding

The `context_text` field is sent to the embedding model (default: `cohere-embed-v4` via the UGA LiteLLM gateway). Embeddings are generated in batches of up to 96 texts with automatic retry and exponential backoff. Cohere's `input_type: search_document` parameter is used for indexing; `search_query` is used at query time for asymmetric search.

### 7. Storage

Embeddings are written as Parquet files to Brightspace course files, one file per module:

```
embeddings/module-{Module Title}.parquet
```

A manifest file tracks indexing state:

```
embeddings/manifest.json
```

## Parquet Schema

Each row in a Parquet file represents one chunk:

| Column | Type | Description |
|--------|------|-------------|
| `chunk_id` | string | Unique identifier (`{topicTitle}-{chunkIndex}`) |
| `module_title` | string | Course module name |
| `topic_title` | string | Document/topic name within the module |
| `topic_type` | string | Source type (`pdf`, `html`, `docx`, `pptx`, `txt`, `md`, `discussion`, `assignment`, `quiz`, `description`, `news`) |
| `chunk_index` | int | 0-based position of this chunk within the topic |
| `total_chunks` | int | Total number of chunks for this topic |
| `chunk_text` | string | Raw source text of the chunk (no context added) |
| `context_text` | string | Enriched text that was embedded (includes module/topic metadata, preamble, and source text with `[CONTEXT TEXT]`/`[SOURCE TEXT]` labels) |
| `embedding` | float[1024] | Vector embedding (1024 dimensions for cohere-embed-v4) |
| `content_hash` | string | SHA-256 hash of the source document bytes |

### Usage Notes for RAG Consumers

- **Search against**: the `embedding` column using cosine similarity
- **Display to user**: the `chunk_text` column (original source text without added context)
- **Send to LLM**: the `context_text` column (includes disambiguating preamble and metadata)
- **Filter by**: `module_title`, `topic_title`, or `topic_type` to scope searches
- **Detect changes**: compare `content_hash` against the current document to check staleness

## Manifest Schema

The `manifest.json` file tracks what has been indexed:

```json
{
  "courseTitle": "Course Name",
  "embeddingModel": "cohere-embed-v4",
  "embeddingDimensions": 1024,
  "chunkSize": 1000,
  "chunkOverlap": 100,
  "modules": [
    {
      "moduleTitle": "Module Name",
      "parquetFile": "module-Module Name.parquet",
      "topicCount": 3,
      "chunkCount": 42,
      "lastIndexed": "2026-03-24T19:43:05.961Z",
      "topicHashes": {
        "Topic Title": "sha256-hash-of-source-document"
      }
    }
  ],
  "lastFullIndex": "2026-03-24T19:43:05.961Z"
}
```

## Settings

| Setting | Default | Description |
|---------|---------|-------------|
| Chunk size | 1000 tokens | Target size for each chunk |
| Chunk overlap | 100 tokens | Overlap between consecutive chunks (rule-based only) |
| Embedding model | `cohere-embed-v4` | Model used for vector generation |
| Embedding provider | `uga-gateway` | LiteLLM gateway identifier |
| Extraction mode | `vision` | `vision` (PDF only) or `programmatic` |
| Chunking mode | `llm-assisted` | `llm-assisted` (two-pass) or `rule-based` (recursive splitter) |
| Chunking instructions | *(empty)* | Free-text instructions appended to chunking prompts |
| Include API content | `true` | Fetch module/topic descriptions, discussion prompts, assignment instructions, quiz instructions, and news from the Brightspace API |

## UI Features

- **Index tab**: Browse modules, drill into individual topics, select items for indexing
- **Topic-level control**: Index, re-index, re-chunk, or delete individual topics within a module
- **Re-chunk**: Re-runs chunking and embedding on cached transcripts without re-extracting source documents
- **Inspect tab**: Browse Parquet file contents, expand full chunk text, view embedded `context_text`, run semantic similarity searches
- **Progress tracking**: Real-time display of pipeline phase, current topic, page progress (for vision extraction), and token usage

## File Structure

```
course-embeddings/
├── components/
│   ├── MainView.tsx          # Primary UI with settings and tab navigation
│   ├── ModuleList.tsx         # Module list with drill-down navigation
│   ├── TopicList.tsx          # Topic-level view with index/delete/re-chunk controls
│   ├── IndexProgress.tsx      # Real-time progress display
│   ├── InspectView.tsx        # Parquet browser and similarity search
│   └── ManifestView.tsx       # Manifest JSON viewer
├── prompts/
│   ├── transcribe-for-embeddings.md   # Vision transcription prompt
│   ├── chunk-boundaries.md            # Pass 1: boundary detection
│   └── chunk-context.md               # Pass 2: context preambles
├── services/
│   ├── indexingPipeline.ts    # Main orchestrator (crawl → extract → chunk → embed → write)
│   ├── contentCrawler.ts      # Brightspace module/topic enumeration
│   ├── visionExtractor.ts     # Vision-based PDF transcription
│   ├── llmChunker.ts          # Two-pass LLM chunking with windowing
│   ├── textChunker.ts         # Rule-based recursive text splitter
│   ├── embeddingClient.ts     # Embedding API client with batching
│   ├── parquetWriter.ts       # Parquet file creation
│   ├── parquetReader.ts       # Parquet file reading and search
│   ├── parquetWasm.ts         # WASM initialization for parquet-wasm
│   ├── similaritySearch.ts    # Cosine similarity computation
│   └── manifestManager.ts    # Manifest CRUD and change detection
├── types.ts                   # TypeScript interfaces and defaults
├── config.ts                  # Agent configuration
├── entry.tsx                  # React entry point
└── entry.html                 # HTML shell
```
