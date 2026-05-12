# TODO: Learning Agent Implementation

## Context

The Learning Agent is a student-facing agent that:
1. Answers questions about course content (using chunked/embedded Parquet data from the course-embeddings agent)
2. Facilitates structured learning activities defined by an `agent-config.md` specification
3. Assesses mastery against defined criteria through conversational interaction
4. Submits a summary, conversation highlights, and full transcript to a Brightspace dropbox on student confirmation

This agent lives inside a Brightspace content topic (iframe) and operates with **student-level permissions** — not instructor. It builds on the prototype at `/home/jcastle/local_dev/brightspace-learning-agent-frontend`.

**Principle**: Where the framework already provides equivalent functionality (chat UI, LLM client, provider settings, agent shell), use it — do not replicate what the prototype does. Where the prototype has functionality the framework lacks, those gaps are tracked in `TODO-CORE-GAPS.md`.

**Prerequisites from `TODO-CORE-GAPS.md`**: This agent depends on several core gaps being filled first:
- Gap 1: Dropbox Submission (core service function)
- Gap 2: Student-Accessible File Loading (`sibling-file` source type)
- Gap 4: Topic-Scoped Context (`useTopicContext` hook)
- Gap 5: Conversation Transcript Tracking (transcript accumulator)
- Gap 6: Voice Mode (core voice services and hook)
- Gap 3: Reusable RAG Service (embedding search — can defer with keyword fallback)

---

## 1. Agent Plugin Definition

Create the Learning Agent using the framework's `defineAgentPlugin` pattern.

```typescript
// agents/learning-agent/config.ts

export default defineAgentPlugin({
  id: 'learning-agent',
  name: 'Learning Agent',
  description: 'Student-facing agent for content Q&A and learning outcome assessment.',
  requiredInputs: [
    {
      key: 'agentConfig',
      label: 'Agent Configuration',
      type: 'markdown',
      sourceType: 'sibling-file',       // NEW core source type
      source: 'agent-config.md',
      required: true,
    },
  ],
});
```

### Work Items
- [ ] Create `agents/learning-agent/config.ts` with plugin definition
- [ ] Create `agents/learning-agent/index.ts` entry point
- [ ] Register in agent catalog

---

## 2. Configuration Parser

Port the `agent-config.md` parser from the prototype. This converts a markdown document with `##` outcome sections into structured data the agent uses for system prompts and dropbox matching.

**From prototype**: `configParser.ts` — `parseAgentConfig()`, `matchDropboxToOutcome()`

### Markdown Schema
```
# General Instructions
Persona and assessment process instructions...

Dropbox Category: Learning Outcomes   (optional, filters dropbox folders)

## Outcome Name 1
Student-facing description (first paragraph).

### Mastery Criteria
- criterion 1
- criterion 2

### Scenarios
- scenario 1

### Tutoring Approach
Prose guidance for tutoring this outcome...

### Dropbox
Explicit Folder Name   (optional, falls back to fuzzy match on outcome name)
```

### Work Items
- [ ] Port `parseAgentConfig()` to `agents/learning-agent/services/configParser.ts`
- [ ] Port `matchDropboxToOutcome()` fuzzy matching logic
- [ ] Port `Outcome` and `OutcomeConfig` TypeScript interfaces
- [ ] Integrate with the framework's `loadInput` to read `agent-config.md` via `sibling-file`

---

## 3. Outcome Selection UI

The prototype presents students with a card-based picker to select which learning outcome to work on. This needs to be built as the agent's primary "landing" view.

**From prototype**: `OutcomeSelector.tsx` — card grid showing `outcome.name` and `outcome.studentDescription`, with onClick to select.

### Work Items
- [ ] Create `agents/learning-agent/components/OutcomeSelector.tsx`
- [ ] Display parsed outcomes as selectable cards
- [ ] Use UGA design system classes (`cmp-card`, `cmp-btn`, `obj-grid`)
- [ ] On selection, transition to the conversational assessment view
- [ ] Handle the case where config has only one outcome (auto-select)

---

## 4. System Prompt Builder

Build the system prompt dynamically based on the selected outcome configuration and retrieved RAG context. This replaces the prototype's `buildSystemPrompt()`.

**Key components of the system prompt**:
- General instructions from config (persona, assessment process)
- Current outcome: name, mastery criteria, scenarios, tutoring approach
- Response style guidance (concise, 2-3 sentences, voice-friendly)
- Conversation opening instructions (start with a scenario, not a vague question)
- Goal assessment instructions (track criteria, summarize when met)
- Submission tool instructions (when and how to call `submit_to_dropbox`)
- RAG context (retrieved course content chunks)

### Work Items
- [ ] Create `agents/learning-agent/services/promptBuilder.ts`
- [ ] Port `buildSystemPrompt()` logic from prototype's `assistantRuntime.ts`
- [ ] Use the framework's prompt templating system (`parseFrontmatter`, `interpolate`) where appropriate
- [ ] Alternatively, store the system prompt as a markdown template in `agents/learning-agent/prompts/system.md` and use the `promptRegistry`
- [ ] Inject RAG context when available

---

## 5. Submission Workflow

Coordinate the submission flow: LLM calls `submit_to_dropbox` tool → agent builds HTML/text → posts to Brightspace dropbox → shows confirmation to student.

**From prototype**:
- `SUBMIT_TOOL` definition in `assistantRuntime.ts` — tool schema for the LLM
- `submissionBuilder.ts` — `buildSubmissionHTML()` and `buildSubmissionText()` that format the submission with header, summary, highlights, and full transcript
- `apiService.ts` `submitToDropbox()` — the actual POST (moving to core per `TODO-CORE-GAPS.md`)

### Work Items
- [ ] Define the `submit_to_dropbox` tool as a custom tool in the agent config (not a framework builtin — it's specific to this agent's workflow)
- [ ] Port `submissionBuilder.ts` to `agents/learning-agent/services/submissionBuilder.ts`
- [ ] Handle tool call processing: parse LLM tool call → build HTML/text → call core `submitToDropbox` service
- [ ] Show submission confirmation dialog before submitting (prototype shows a card with summary that the student confirms)
- [ ] Display success/error feedback after submission
- [ ] Match outcome to dropbox folder using `matchDropboxToOutcome()` (or explicit `dropboxName` from config)
- [ ] If a dropbox category is specified in config, filter folders to that category first

---

## 6. Conversational Assessment View

The main chat view for the learning interaction. Uses the framework's existing chat UI components but adds agent-specific elements.

### Work Items
- [ ] Create `agents/learning-agent/components/AssessmentChat.tsx`
- [ ] Wire up the framework's chat components (message list, input, send button)
- [ ] Add the voice mode toggle button (uses core `useVoiceMode` hook from `TODO-CORE-GAPS.md`)
- [ ] Display interim transcript when in voice mode
- [ ] Connect LLM streaming `onChunk` to voice mode's `pushTextDelta` for streaming TTS
- [ ] Add outcome context display (banner showing which outcome is active)
- [ ] Add "Back to outcomes" navigation
- [ ] Handle the `submit_to_dropbox` tool call in the agent loop

---

## 7. RAG Integration

Connect the agent to course content embeddings for context-aware responses.

### Work Items
- [ ] On agent load, check for `embeddings/manifest.json` in the course files
- [ ] If embeddings exist, use core `searchCourseEmbeddings()` (from `TODO-CORE-GAPS.md`) to retrieve relevant chunks for each user message
- [ ] If no embeddings, fall back gracefully (respond based on prompt context only, or use keyword search as in the prototype)
- [ ] Scope RAG retrieval to the root module of the current topic (using `useTopicContext()`)
- [ ] Inject top-K results into the system prompt as `--- Retrieved Course Content ---` block
- [ ] Consider: should the agent tell the student when it doesn't have content to reference?

---

## 8. Content Indexing (Optional / Deferred)

The prototype includes client-side content indexing (`contentIndexer.ts`) that crawls the TOC, extracts text from HTML/PDF/DOCX, chunks it, and stores it in IndexedDB. This is a fallback for when course-embeddings Parquet files don't exist.

**Decision needed**: Is this needed at all if we have the course-embeddings agent generating Parquet files? If yes, it could be a framework-level utility rather than agent-specific.

### Work Items (if proceeding)
- [ ] Evaluate whether the course-embeddings pipeline is sufficient or if client-side indexing is needed as fallback
- [ ] If needed, port `contentIndexer.ts` and `ragStore.ts` to core or to this agent
- [ ] If not needed, ensure the agent degrades gracefully when no embeddings are available

---

## 9. Agent Deployment Configuration

The Learning Agent has special deployment needs since it runs in student context.

### Work Items
- [ ] Document the deployment process: build → upload to course files → insert as content topic
- [ ] Ensure `agent-config.md` is placed alongside the deployed HTML
- [ ] Ensure VAD assets (`silero_vad_legacy.onnx`, `vad.worklet.bundle.min.js`) are bundled or CDN-loaded
- [ ] Document how to set up dropbox folders that match outcome names
- [ ] Document the `Dropbox Category` config option for filtering
- [ ] Test that the agent works without Manage Files permissions (student role)

---

## 10. Testing Checklist

- [ ] Agent loads `agent-config.md` from sibling file without instructor permissions
- [ ] Config parses into correct outcomes with criteria, scenarios, and tutoring approach
- [ ] Outcome selection UI renders all outcomes and allows selection
- [ ] System prompt correctly incorporates selected outcome and RAG context
- [ ] Conversational flow is natural (2-3 sentences, scenario-driven opening, criteria tracking)
- [ ] Voice mode: STT captures speech, TTS reads responses, barge-in interrupts TTS
- [ ] RAG retrieval returns relevant course content and injects it into prompt
- [ ] `submit_to_dropbox` tool is called only after student confirms
- [ ] Submission contains formatted HTML with summary, highlights, and transcript
- [ ] Submission posts successfully to the matched dropbox folder
- [ ] Agent degrades gracefully when embeddings are not available
- [ ] Agent works in iframe context (topic-scoped)
- [ ] Agent works in dev mode (direct access, no iframe)

---

## Implementation Order

1. **Agent plugin scaffolding** (config, entry point, catalog registration) — no core dependencies
2. **Configuration parser** — no core dependencies, pure TypeScript
3. **Outcome selection UI** — needs config parser only
4. **System prompt builder** — needs config parser only
5. **Submission builder** — needs config parser and transcript accumulator (core Gap 5)
6. **Core gaps**: Dropbox submission, sibling-file loading, topic context — parallel with above
7. **Submission workflow** — needs core dropbox submission + submission builder
8. **RAG integration** — needs core RAG service (Gap 3) or can start with keyword fallback
9. **Conversational assessment view** — ties everything together
10. **Voice mode integration** — needs core voice services (Gap 6)
11. **Deployment and testing**
