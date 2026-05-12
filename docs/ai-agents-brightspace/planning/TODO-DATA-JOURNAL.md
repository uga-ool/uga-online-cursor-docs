# TODO: Journal / Notes System

## Context

The ID Assistant needs to record unstructured information from different users across sessions. When a designer notes that an SME expects Module 3 by April 10, or a faculty member mentions IRB approval came through, the agent should store that and be able to recall it later.

The journal uses JSON as the source of truth and (eventually) Parquet embeddings as a semantic search index over the entries.

## What Exists Today

- `readCourseFile` / `uploadFileToCourse` — reads and writes JSON to course files
- `enqueueSave` pattern — serializes concurrent writes within a single browser session (used by course-review and state persistence)
- Course-embeddings agent — full embedding pipeline (crawl → chunk → embed → Parquet) with in-browser cosine similarity search
- `whoAmI.Identifier` — identifies who is writing each entry

## Work Items

### 1. Journal Entry Schema

See `TODO-DATA-SCHEMAS.md` for the full TypeScript interface. Core fields:

```
id:        string    — unique entry ID (UUID or nanoid)
timestamp: string    — ISO 8601
userId:    string    — whoAmI.Identifier
userName:  string    — display name (denormalized)
userRole:  string    — role from the program registry (designer, sme, coordinator)
courseOu:  string    — which course this relates to (optional, may be program-wide)
tags:      string[]  — lightweight categorization
content:   string    — the actual note (free-form text)
source:    string    — how the entry was created: "user-input", "agent-inferred", "conversation"
```

### 2. Per-User Journal Files

Use per-user files to avoid cross-user write conflicts:

```
id-assistant/journal/
├── journal_98765.json    ← Jane's entries
├── journal_11111.json    ← Dr. Jones's entries
└── journal_22222.json    ← Sarah's entries
```

- Each user writes only to their own file (`journal_{whoAmI.Identifier}.json`)
- The agent reads all journal files on startup and merges them in memory
- `listCourseFolder(ou, 'id-assistant/journal')` to discover all journal files
- No concurrent write conflicts possible since each user owns their file

### 3. Writing Journal Entries

Two mechanisms for creating entries:

**Agent tool (`save_note`)**: The LLM calls this during conversation to record information.
- "I'll note that Dr. Jones expects Module 3 materials by April 10."
- The agent auto-populates `userId`, `userName`, `userRole`, `timestamp`
- The LLM provides `content`, `courseOu` (if applicable), and optionally `tags`

**UI input**: A form in the agent where users can directly add notes.
- Useful for structured data entry outside of conversation
- Same underlying write path

### 4. Reading / Retrieving Journal Entries

Three tiers, implement progressively:

**Tier 1 — Context injection (start here)**:
- On each conversation, load all journal files, filter by current user's courses and/or recent timeframe
- Inject relevant entries into the system prompt
- Works when journal is small (< ~10-20k tokens across all entries)

**Tier 2 — Structured search tool (`search_notes`)**:
- The LLM calls a tool to filter entries by `courseOu`, `userId`, `tags`, date range
- Array filtering on the merged in-memory journal
- No embedding required — just field-based lookups

**Tier 3 — Semantic search (when needed)**:
- Embed journal entries using the course-embeddings pipeline
- Store embeddings in Parquet alongside course content embeddings
- Use cosine similarity to find relevant entries by meaning
- The JSON files remain the source of truth; Parquet is a regenerable search index

### 5. Separate Raw Content from Inferred Data

When the agent extracts structured information from a note (a deadline, a person, a module), store the extraction separately:

```json
{
  "content": "Dr. Jones says Module 3 won't be ready until mid-April.",
  "extracted": {
    "deadline": "2026-04-15",
    "person": "Dr. Jones",
    "module": "Module 3",
    "status": "delayed"
  }
}
```

The `extracted` field is optional and can be re-derived later with a better model. The `content` field is the immutable human record.

## Design Decisions to Make

- [ ] Should journal entries be append-only (immutable once written) or editable?
- [ ] Should users be able to see each other's journal entries, or only entries relevant to their courses?
- [ ] Should the agent auto-tag entries, or only use user-provided tags?
- [ ] At what journal size should Tier 2/3 retrieval be triggered? (Estimate based on typical usage patterns)
- [ ] Should there be a journal entry retention policy, or keep everything indefinitely?
