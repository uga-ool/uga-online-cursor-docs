# TODO: Core Framework Gaps (for Student-Facing Agents)

## Context

The Learning Agent prototype reveals several framework-level capabilities that are missing from core. These gaps are not specific to one agent — any student-facing agent deployed inside Brightspace content would encounter them. Addressing these in core makes them reusable across all agents.

Reference prototype: `/home/jcastle/local_dev/brightspace-learning-agent-frontend`

## Gap 1: Dropbox Submission (POST)

**Problem**: The framework has read-only dropbox tools (`get_dropbox_folders`, `get_dropbox_folder`, `get_dropbox_categories`, `get_dropbox_category`, `get_dropbox_submissions`, `get_dropbox_feedback`) but **no write/submit capability**.

**What the prototype does** (`apiService.ts`):
- `POST` to `/d2l/api/le/{version}/{ou}/dropbox/folders/{folderId}/submissions/mysubmissions/`
- Multipart/mixed body with:
  - Part 1: JSON comment (`{ Text: plainText, Html: html }`)
  - Part 2: A placeholder text file (Brightspace requires at least one file part)
- Headers: `Content-Type: multipart/mixed; boundary=...`, `X-Csrf-Token`, `credentials: 'include'`

**What to add to core**:

Option A — A new function in `core/brightspace/` (like `uploadFileToCourse`):
```typescript
// core/brightspace/dropbox.ts
export async function submitToDropbox(
  ou: string,
  folderId: number,
  xsrfToken: string,
  comment: { text: string; html: string },
  file?: { name: string; content: Blob | string; contentType?: string }
): Promise<void>
```

Option B — A new tool in `builtins/dropbox.ts` that the LLM can call:
```typescript
{
  name: 'submit_to_dropbox',
  description: 'Submit work to an assignment dropbox folder.',
  category: 'dropbox',
  service: 'le',
  pathTemplate: '/{ou}/dropbox/folders/{folderId}/submissions/mysubmissions/',
  method: 'POST',  // new: current tools are all GET
  ...
}
```

**Recommendation**: Both. Add the service function (Option A) for programmatic use by agents, and optionally a tool definition (Option B) for LLM-driven submission. The service function is the priority — the LLM tool can wrap it.

**Note**: This is a write operation using the *student's* session. The student submitting their own work to their own dropbox is a permitted Brightspace operation — it doesn't require instructor permissions. But the tool executor currently only supports GET requests. Supporting POST requires extending the executor or keeping submission as a dedicated service function outside the tool system.

### Work Items
- [ ] Create `core/brightspace/dropbox.ts` with `submitToDropbox` function
- [ ] Handle multipart/mixed body construction (JSON comment + file part)
- [ ] Export from `@uga-brightspace/framework` public API
- [ ] Add TypeScript types for submission comment and response
- [ ] Consider whether to also expose as an LLM tool definition

---

## Gap 2: Student-Accessible File Loading

**Problem**: The framework's `readCourseFile` uses the Manage Files API (`/{ou}/managefiles/file`), which requires instructor-level permissions. Students can't use it. But agents deployed inside content topics need to load configuration files.

**What the prototype does** (`CourseContext.tsx`):
- Loads `agent-config.md` from a URL **sibling to `index.html`** — the same folder where the agent is deployed in course files
- Uses plain `fetch(url, { credentials: 'include' })` — no Manage Files API needed
- The URL is constructed relative to the current page: the agent figures out its own deployment path and loads the config from the same location

**What to add to core**:

A new data loading function and/or source type for reading files relative to the agent's deployment location:

```typescript
// core/brightspace/files.ts or core/context/dataLoader.ts

/** Load a file from the same folder as the agent's HTML entry point. */
export async function loadSiblingFile(
  relativePath: string,
  type: 'json' | 'markdown' | 'text'
): Promise<unknown>
```

Or a new `sourceType` for `InputDeclaration`:
```typescript
{
  key: 'agentConfig',
  type: 'markdown',
  sourceType: 'sibling-file',  // NEW: relative to agent's deployed location
  source: 'agent-config.md',
  required: true,
}
```

### Work Items
- [ ] Implement a `loadSiblingFile` function that constructs a URL relative to `document.baseURI` or `import.meta.url`
- [ ] Add `sibling-file` as a new `DataSourceType` in `shared/src/types/plugin.ts`
- [ ] Update `dataLoader.ts` to handle the new source type
- [ ] Export from `@uga-brightspace/framework` public API
- [ ] Document when to use `sibling-file` vs `lms-file` (student context vs instructor context)

---

## Gap 3: Reusable RAG Service

**Problem**: The course-embeddings agent generates Parquet files with vector embeddings, but there's no framework-level API for other agents to consume them. The prototype has its own client-side RAG (IndexedDB + keyword search), which should be replaced by the real embedding infrastructure.

**What exists in the framework** (course-embeddings agent, not in core):
- `parquetReader.ts` — reads Parquet files from course files using WASM
- `similaritySearch.ts` — cosine similarity over `ChunkRow[]` with `number[]` embeddings
- `embeddingClient.ts` — generates embeddings via the LiteLLM gateway (Cohere embed-v4)
- `manifestManager.ts` — reads the `embeddings/manifest.json` to know what's been indexed

**What the prototype does** (`ragStore.ts`, `contentIndexer.ts`):
- Client-side indexing: crawl TOC → fetch content → extract text → chunk → store in IndexedDB
- Keyword-based retrieval (word overlap scoring, no embeddings)
- Scoped by `ou:rootModuleId`

**What to add to core**:

A reusable RAG service that loads Parquet embeddings from course files and performs similarity search:

```typescript
// core/rag/search.ts

export interface RAGSearchResult {
  chunkText: string;
  contextText: string;
  moduleTitle: string;
  topicTitle: string;
  topicType: string;
  score: number;
}

/** Load embeddings from a course's Parquet files and search by query. */
export async function searchCourseEmbeddings(
  ou: string,
  query: string,
  options?: {
    topK?: number;
    moduleFilter?: string;
    topicFilter?: string;
  }
): Promise<RAGSearchResult[]>
```

This would:
1. Read `embeddings/manifest.json` from the target course to know which Parquet files exist
2. Load the relevant Parquet files via `readCourseFile` (or `fetchCourseFileRaw` for binary)
3. Generate a query embedding via the embedding client
4. Run cosine similarity search
5. Return the top-K results

### Work Items
- [ ] Extract Parquet reading, similarity search, and embedding client from course-embeddings into `core/rag/`
- [ ] Create `searchCourseEmbeddings` function with manifest-aware loading
- [ ] Handle the WASM Parquet decoder initialization (currently in `parquetWasm.ts`)
- [ ] Support cross-course RAG (pass target `ou` for the hub model)
- [ ] Cache loaded embeddings in memory to avoid re-reading Parquet files on every query
- [ ] Export from `@uga-brightspace/framework` public API
- [ ] Consider an LLM tool wrapper: `search_course_content(query, ou?, module?)`

**Note**: The course-embeddings agent currently embeds using `input_type: search_document` for indexing and `search_query` for queries (Cohere asymmetric search). The RAG service needs to use the correct input type for queries.

---

## Gap 4: Topic-Scoped Context

**Problem**: Agents deployed inside a content topic (via iframe) need to know which topic they're embedded in and scope their view to the containing module. The framework's `BrightspaceProvider` extracts `ou` from the URL but doesn't extract `topicId` or determine the root module.

**What the prototype does** (`CourseContext.tsx`):
- Checks if running in an iframe (`window.self !== window.top`)
- Extracts `topicId` from the parent URL: `/d2l/le/content/{ou}/topics/{topicId}/View`
- Walks the TOC tree to find which root module contains that topic
- Scopes content indexing to only topics within that root module
- Skips the agent's own topic (to avoid indexing itself)

**What to add to core**:

Extend `BrightspaceProvider` or add a companion hook:

```typescript
// core/brightspace/context.tsx or core/brightspace/topicContext.ts

export interface TopicContext {
  topicId: string | null;
  rootModuleId: number | null;
  rootModuleTitle: string | null;
  isEmbedded: boolean;          // true if running in iframe
  siblingTopics: MappedTopic[]; // other topics in the same module
}

export function useTopicContext(): TopicContext;
```

### Work Items
- [ ] Add iframe detection and parent URL parsing for `topicId`
- [ ] Add TOC tree walker to resolve root module from topic ID
- [ ] Expose as `useTopicContext()` hook or extend `useBrightspace()` return value
- [ ] Export `TopicContext` type and hook from `@uga-brightspace/framework`
- [ ] Handle the case where the agent is NOT in an iframe (direct access, dev mode)

---

## Gap 5: Conversation Transcript Tracking

**Problem**: The framework's LLM client streams responses but doesn't maintain a structured conversation transcript that agents can access for purposes beyond the LLM call (e.g., building a dropbox submission with the full conversation history).

**What the prototype does** (`ChatAssistant.tsx`):
- Maintains `conversationHistory` as `ConversationMessage[]` (`{ role, text }`)
- Updated via `onMessage` callback after each complete exchange
- Passed to `submissionBuilder` to generate the submission HTML/text

**What to add to core**:

A lightweight transcript accumulator, either in the LLM client or as a utility:

```typescript
// core/llm/transcript.ts

export interface TranscriptEntry {
  role: 'user' | 'assistant';
  content: string;
  timestamp: string;
}

export interface TranscriptAccumulator {
  entries: TranscriptEntry[];
  addUserMessage(content: string): void;
  addAssistantMessage(content: string): void;
  getTranscript(): TranscriptEntry[];
  clear(): void;
  toText(): string;
  toHTML(): string;
}

export function createTranscriptAccumulator(): TranscriptAccumulator;
```

### Work Items
- [ ] Create `core/llm/transcript.ts` with accumulator
- [ ] Include `toText()` and `toHTML()` formatters for common output needs
- [ ] Export from `@uga-brightspace/framework`
- [ ] Consider integration with the agent loop so transcript accumulates automatically

---

## Gap 6: Voice Mode

**Problem**: The framework has no voice capabilities. The prototype has a full voice pipeline: STT (Web Speech API), TTS (OpenAI API), and VAD (Voice Activity Detection) for barge-in.

**What the prototype has**:

**`voiceService.ts`** — Two standalone functions + a `TTSQueue` class:
- `transcribeAudio(apiKey, audioBlob)` — Whisper transcription via OpenAI API (unused in main flow but available)
- `speakText(apiKey, text)` — OpenAI TTS (`tts-1`, voice `nova`, MP3 format), returns `HTMLAudioElement`
- `TTSQueue` — Buffers streaming text deltas, splits at sentence boundaries, pre-fetches TTS audio per sentence, plays sequentially. Supports `push(delta)`, `flush()`, `cancel()`, `reset()`. Callbacks: `onSpeakingChange`, `onFinished`.

**`useVoiceMode.ts`** — React hook managing the full lifecycle:
- **STT**: Browser `SpeechRecognition` API (continuous, interim results). Final segments debounced with 2s pause before sending.
- **TTS**: `TTSQueue` fed by `pushTextDelta` from the LLM stream's `onChunk` callback.
- **VAD**: `@ricky0123/vad-web` with Silero ONNX model. Detects user speech during TTS playback → cancels TTS (barge-in) → resumes speech recognition.
- **Echo suppression**: Recognition paused while TTS plays to avoid the agent hearing itself. VAD runs on a separate stream with echo cancellation.

**What to add to core**:

### 6a. Voice Services (`core/voice/`)

```
core/voice/
├── tts.ts          ← speakText, TTSQueue (from voiceService.ts)
├── stt.ts          ← SpeechRecognition wrapper, transcribeAudio
├── vad.ts          ← VAD wrapper (dynamic import of @ricky0123/vad-web)
└── types.ts        ← TTSConfig, STTConfig, VoiceState
```

**TTS Service** (`tts.ts`):
- `speakText(apiKey, text, options?)` — generate speech via OpenAI API
- `TTSQueue` class — sentence-boundary buffering, sequential playback, cancel/reset
- Configurable: model (`tts-1`, `tts-1-hd`), voice (`nova`, `alloy`, `echo`, `fable`, `onyx`, `shimmer`), response format
- Provider-agnostic: the TTS endpoint URL should be configurable (not hardcoded to OpenAI) to support UGA gateway or other providers

**STT Service** (`stt.ts`):
- `isSpeechRecognitionSupported` — browser capability check
- `createSpeechRecognition(options)` — wrapper around `SpeechRecognition` API with continuous mode, interim results, auto-restart
- `transcribeAudio(apiKey, audioBlob)` — Whisper API transcription (alternative to browser STT)

**VAD Service** (`vad.ts`):
- `createVAD(options)` — wrapper around `@ricky0123/vad-web` with dynamic import
- Handles ONNX WASM loading from CDN
- `onSpeechStart` / `onSpeechEnd` callbacks for barge-in detection

### 6b. Voice Mode Hook (`core/voice/useVoiceMode.ts`)

The React hook from the prototype, generalized:

```typescript
export interface UseVoiceModeOptions {
  apiKey: string;
  onSendMessage: (text: string) => void;
  ttsConfig?: { model?: string; voice?: string; provider?: string };
  sendDelayMs?: number;   // debounce delay (default 2000ms)
}

export interface UseVoiceModeReturn {
  isSupported: boolean;
  isActive: boolean;
  isListening: boolean;
  isSpeaking: boolean;
  interimTranscript: string;
  toggleVoiceMode: () => void;
  pushTextDelta: (delta: string) => void;
  flushTTS: () => void;
  resetTTS: () => void;
  cancelTTS: () => void;
}
```

### 6c. Dependencies and Assets

- `@ricky0123/vad-web` — add to framework's package.json (or keep as optional peer dependency)
- `onnxruntime-web` — required by VAD (loaded from CDN in prototype)
- VAD model assets (`silero_vad_legacy.onnx`, `vad.worklet.bundle.min.js`) — need to be served from the agent's `public/` directory or loaded from CDN
- Consider: should VAD assets be bundled with the framework or loaded from CDN at runtime?

### 6d. Integration with LLM Streaming

The framework's `callLLMStream` already has an `onChunk` callback. The voice integration point:
- Connect `onChunk` → `ttsQueue.push(delta)` for streaming TTS
- On stream complete → `ttsQueue.flush()`
- On new user message → `ttsQueue.cancel()` (or `reset()`)

### Work Items
- [ ] Create `core/voice/tts.ts` — `speakText` function and `TTSQueue` class
- [ ] Create `core/voice/stt.ts` — `SpeechRecognition` wrapper
- [ ] Create `core/voice/vad.ts` — VAD wrapper with dynamic import
- [ ] Create `core/voice/useVoiceMode.ts` — React hook
- [ ] Create `core/voice/types.ts` — shared types
- [ ] Add `@ricky0123/vad-web` to dependencies (or optional peer deps)
- [ ] Determine asset strategy for VAD ONNX model (CDN vs bundled)
- [ ] Make TTS provider configurable (OpenAI, UGA gateway, etc.)
- [ ] Export all voice utilities from `@uga-brightspace/framework`
- [ ] Add voice toggle UI component to the framework's chat CSS classes

---

## Priority Order

1. **Dropbox Submission** — Blocks the learning agent's core use case (can't submit work without it)
2. **Student-Accessible File Loading** — Blocks config loading in student context
3. **Topic-Scoped Context** — Needed for proper content scoping in iframe deployments
4. **Conversation Transcript** — Small addition, needed for submission builder
5. **Reusable RAG Service** — Important but prototype has keyword fallback; can iterate
6. **Voice Mode** — Large feature area but not a blocker for core functionality

## Design Decisions to Make

- [ ] Should dropbox submission be a tool the LLM can call directly, or only a service function agents invoke programmatically?
- [ ] Should `sibling-file` loading work only for the agent's own folder, or support relative paths to parent/sibling folders?
- [ ] Should the RAG service cache Parquet data in IndexedDB (like the prototype) or only in memory?
- [ ] Should voice mode support provider-agnostic TTS (UGA gateway, ElevenLabs, etc.) or start with OpenAI only?
- [ ] Should VAD assets be bundled or loaded from CDN? (Tradeoff: bundle size vs. reliability)
- [ ] Should the transcript accumulator auto-integrate with the agent loop, or be opt-in per agent?
