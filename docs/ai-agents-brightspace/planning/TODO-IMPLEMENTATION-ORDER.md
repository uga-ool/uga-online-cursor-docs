# TODO: Implementation Order

## Overview

This document defines the recommended implementation sequence for the ID Assistant, Learning Agent, and related framework enhancements. Items are ordered by dependency — each phase builds on the previous one. Complexity ratings are relative to each other.

---

## Phase 1: Foundations

### 1. Data Schemas (Medium complexity)
**Prerequisite for**: Registry, Journal, Skills, Identity (allowlist)
**File**: `TODO-DATA-SCHEMAS.md`

Define TypeScript interfaces in `shared/src/types/` and Zod validation schemas in `shared/src/schemas/`. This is the contract that everything else depends on. Getting it wrong means painful migrations later.

- [ ] `ProgramRegistry`, `ProgramCourse`, `TeamMember`, `ProgramTeam`
- [ ] `JournalFile`, `JournalEntry`
- [ ] `SkillManifest`, `SkillFile`, `AgentSkillsConfig`
- [ ] `AccessControl`
- [ ] Export from `shared/src/index.ts`
- [ ] Add Zod schemas for runtime validation on load

### 2. Identity & Access Control (Low complexity)
**Prerequisite for**: Registry (registry-as-access-control)
**Depends on**: Schemas (for `AccessControl` type)
**File**: `TODO-IDENTITY.md`

Mostly leveraging existing `whoAmI` / `courseRole` infrastructure. Small, focused additions.

- [ ] Role-based access check pattern (agent-level, potentially a `<RequireRole>` wrapper)
- [ ] Allowlist-based access via `{agent}/access.json` loaded as `lms-file`
- [ ] Graceful access-denied UI

### 3. Cross-Course Data Loading (Low-Medium complexity)
**Prerequisite for**: Registry (cross-course reads)
**Depends on**: Nothing (existing APIs are sufficient)
**File**: `TODO-DATA-LOADING.md`

The plumbing already works. This is about making it explicit and usable by the LLM.

- [ ] Add optional `ou` parameter to tool definitions in `course.ts` (and any other relevant categories)
- [ ] Improve error handling in `readCourseFile` for permission failures vs. missing files
- [ ] Document cross-course tool usage patterns for system prompts

---

## Phase 2: Program Hub Core

### 4. Program Registry (Medium complexity)
**Prerequisite for**: Journal (team context), Skills (program-level storage)
**Depends on**: Schemas, Identity, Data Loading
**File**: `TODO-DATA-REGISTRY.md`

The central data structure for the ID Assistant. Enables the agent to know which courses and people it's working with.

- [ ] Registry data service (load/save `id-assistant/program.json`)
- [ ] Admin UI for adding/removing courses (validates via `fetchCourseDetails`)
- [ ] Admin UI for assigning team members
- [ ] Offering discovery via `fetchOrgUnitChildren` for registered templates
- [ ] System prompt construction with registered course context
- [ ] Match `whoAmI.Identifier` against registry for role-aware behavior

### 5. Journal / Notes System — Tier 1 & 2 (Medium complexity)
**Prerequisite for**: Nothing downstream (Tier 3 is independent and can be added later)
**Depends on**: Schemas, Registry (for user role context)
**File**: `TODO-DATA-JOURNAL.md`

Start with Tier 1 (context injection) and Tier 2 (structured search). Skip Tier 3 (semantic search) until the journal grows large enough to need it.

- [ ] Per-user journal files (`id-assistant/journal/journal_{userId}.json`)
- [ ] `save_note` tool for the LLM to record information during conversation
- [ ] `search_notes` tool for filtered lookups (by course, user, tags, date range)
- [ ] Journal loading and merging on startup (read all `journal_*.json` files)
- [ ] Context injection — filter relevant entries into system prompt
- [ ] UI for viewing/managing notes (optional at this stage)

---

## Phase 3: Core Gaps for Student-Facing Agents

New capabilities required by any student-facing agent (Learning Agent and beyond). These are framework-level additions.

### 6. Dropbox Submission Service (Low-Medium complexity)
**Prerequisite for**: Learning Agent submission workflow
**Depends on**: Nothing (uses existing `xsrfToken` and Brightspace session)
**File**: `TODO-CORE-GAPS.md` (Gap 1)

The framework has read-only dropbox tools but no write capability. Adds a core service function for posting submissions via the multipart/mixed Brightspace API.

- [ ] Create `core/brightspace/dropbox.ts` with `submitToDropbox` function
- [ ] Handle multipart/mixed body construction (JSON comment + file part)
- [ ] Export from `@uga-brightspace/framework` public API
- [ ] Add TypeScript types for submission comment and response

### 7. Student-Accessible File Loading (Low complexity)
**Prerequisite for**: Learning Agent config loading, any agent deployed in student context
**Depends on**: Nothing
**File**: `TODO-CORE-GAPS.md` (Gap 2)

Students can't use the Manage Files API. Adds a `sibling-file` source type that loads files relative to the agent's deployment location via plain `fetch`.

- [ ] Implement `loadSiblingFile` function (URL relative to `document.baseURI`)
- [ ] Add `sibling-file` as a new `DataSourceType` in `shared/src/types/plugin.ts`
- [ ] Update `dataLoader.ts` to handle the new source type
- [ ] Export from `@uga-brightspace/framework`

### 8. Topic-Scoped Context (Low-Medium complexity)
**Prerequisite for**: Learning Agent content scoping, RAG module filtering
**Depends on**: Nothing (uses existing TOC API)
**File**: `TODO-CORE-GAPS.md` (Gap 4)

Agents embedded in content topics need to know which topic/module they're in. Adds iframe detection, `topicId` extraction from parent URL, and root module resolution.

- [ ] Add iframe detection and parent URL parsing for `topicId`
- [ ] Add TOC tree walker to resolve root module from topic ID
- [ ] Expose as `useTopicContext()` hook or extend `useBrightspace()` return value
- [ ] Export `TopicContext` type and hook from `@uga-brightspace/framework`
- [ ] Handle non-iframe cases (direct access, dev mode)

### 9. Conversation Transcript Tracking (Low complexity)
**Prerequisite for**: Learning Agent submission builder
**Depends on**: Nothing
**File**: `TODO-CORE-GAPS.md` (Gap 5)

Lightweight accumulator that tracks user/assistant exchanges with formatters for submission output.

- [ ] Create `core/llm/transcript.ts` with `TranscriptAccumulator`
- [ ] Include `toText()` and `toHTML()` formatters
- [ ] Export from `@uga-brightspace/framework`
- [ ] Consider integration with the agent loop for automatic accumulation

### 10. Voice Mode (Medium-High complexity)
**Prerequisite for**: Learning Agent voice interaction
**Depends on**: Nothing (standalone capability, integrates with existing `callLLMStream` `onChunk`)
**File**: `TODO-CORE-GAPS.md` (Gap 6)

Full voice pipeline in `core/voice/`: TTS (OpenAI API, sentence-boundary buffering, sequential playback), STT (Web Speech API wrapper), VAD (barge-in detection via `@ricky0123/vad-web`), and the `useVoiceMode` React hook.

- [ ] Create `core/voice/tts.ts` — `speakText` function and `TTSQueue` class
- [ ] Create `core/voice/stt.ts` — `SpeechRecognition` wrapper
- [ ] Create `core/voice/vad.ts` — VAD wrapper with dynamic import
- [ ] Create `core/voice/useVoiceMode.ts` — React hook
- [ ] Create `core/voice/types.ts` — shared types and config
- [ ] Add `@ricky0123/vad-web` to dependencies (or optional peer deps)
- [ ] Determine asset strategy for VAD ONNX model (CDN vs bundled)
- [ ] Make TTS provider configurable (OpenAI, UGA gateway, etc.)
- [ ] Export all voice utilities from `@uga-brightspace/framework`
- [ ] Add voice toggle UI component to framework chat CSS classes

### 11. Reusable RAG Service (Medium complexity)
**Prerequisite for**: Learning Agent content Q&A, any agent consuming course embeddings
**Depends on**: course-embeddings agent Parquet output
**File**: `TODO-CORE-GAPS.md` (Gap 3)

Extract the Parquet reader, embedding client, and similarity search from the course-embeddings agent into `core/rag/` so any agent can consume embedded course content.

- [ ] Extract Parquet reading, similarity search, and embedding client into `core/rag/`
- [ ] Create `searchCourseEmbeddings` function with manifest-aware loading
- [ ] Handle WASM Parquet decoder initialization
- [ ] Support cross-course RAG (pass target `ou` for the hub model)
- [ ] Cache loaded embeddings in memory to avoid re-reading Parquet files per query
- [ ] Export from `@uga-brightspace/framework`
- [ ] Consider an LLM tool wrapper: `search_course_content(query, ou?, module?)`

---

## Phase 4: Learning Agent

Depends on Phase 3 core gaps. Pure agent-level work — no further framework changes needed.

### 12. Agent Scaffolding & Config Parser (Low complexity)
**Depends on**: Sibling-file loading (#7)
**File**: `TODO-LEARNING-AGENT.md` (sections 1–2)

Create the agent plugin definition and port the `agent-config.md` parser from the prototype.

- [ ] `defineAgentPlugin` with `sibling-file` input for `agent-config.md`
- [ ] Port `parseAgentConfig()` and `matchDropboxToOutcome()`
- [ ] Port `Outcome` and `OutcomeConfig` interfaces

### 13. Outcome Selection & System Prompt (Low-Medium complexity)
**Depends on**: Config parser (#12)
**File**: `TODO-LEARNING-AGENT.md` (sections 3–4)

Outcome card picker UI and dynamic system prompt builder.

- [ ] Outcome selector component using UGA design system
- [ ] System prompt builder (outcome-aware, RAG-injected)
- [ ] Use framework prompt templating where appropriate

### 14. Submission Workflow (Medium complexity)
**Depends on**: Dropbox submission service (#6), transcript tracking (#9), config parser (#12)
**File**: `TODO-LEARNING-AGENT.md` (section 5)

End-to-end submission: LLM tool call → builder → dropbox POST → student confirmation.

- [ ] `submit_to_dropbox` custom tool definition
- [ ] Port `submissionBuilder.ts` (HTML/text formatters with transcript)
- [ ] Match outcome to dropbox folder (explicit name or fuzzy match)
- [ ] Submission confirmation dialog
- [ ] Success/error feedback

### 15. Conversational Assessment View (Medium complexity)
**Depends on**: Outcome selection (#13), submission workflow (#14), voice mode (#10)
**File**: `TODO-LEARNING-AGENT.md` (section 6)

The main chat view. Uses framework chat UI components, adds voice toggle and outcome context.

- [ ] Wire up framework chat components with agent-specific elements
- [ ] Voice mode toggle and interim transcript display
- [ ] Connect streaming `onChunk` → `pushTextDelta` for TTS
- [ ] Handle `submit_to_dropbox` tool call in agent loop
- [ ] Outcome context banner and navigation

### 16. RAG Integration (Low-Medium complexity)
**Depends on**: Reusable RAG service (#11), topic-scoped context (#8)
**File**: `TODO-LEARNING-AGENT.md` (section 7)

Connect the agent to course content embeddings, scoped to the active module.

- [ ] Check for `embeddings/manifest.json` on load
- [ ] Use core `searchCourseEmbeddings()` for each user message
- [ ] Scope retrieval to root module via `useTopicContext()`
- [ ] Inject top-K results into system prompt
- [ ] Graceful degradation when no embeddings are available

### 17. Deployment & Testing (Low complexity)
**Depends on**: All above
**File**: `TODO-LEARNING-AGENT.md` (sections 9–10)

- [ ] Document deployment process (build → upload → insert as content topic)
- [ ] Ensure agent works without Manage Files permissions (student role)
- [ ] Verify VAD assets are available (CDN or bundled)
- [ ] Document dropbox folder naming and category conventions
- [ ] Full testing checklist (config loading, outcomes, voice, RAG, submission)

---

## Independent Track: Course Export Agent Completion

### Course Export Wizard — Reverse Engineering & Completion (Medium complexity)
**Prerequisite for**: Agent Orchestration (the export agent must work before it can be delegated to)
**Depends on**: Nothing (in-progress agent, standalone work)
**File**: `TODO-COURSE-EXPORT-WIZARD.md`

The Course Export Agent scaffold, UI, orchestrator, and session keep-alive are implemented. The remaining work requires hands-on investigation of the Brightspace export wizard's HTTP flow using browser DevTools. This is empirical work that cannot be done without access to a live Brightspace instance.

- [ ] DevTools capture: document the 3-step wizard form submissions (field names, values, POST targets)
- [ ] Verify/correct form field names for Step 1 (component selection) and Step 2 (confirmation + "Include Course Files")
- [ ] Determine whether unchecked tools omit their fields or send `false`
- [ ] Identify the notification/alerts endpoint that delivers the export completion signal
- [ ] Decode the download URL structure (`fid` = `{GUID};{filename};{number}`, numeric path segment = .NET tick count or file ID)
- [ ] Implement `checkExportStatus()` in `jobPoller.ts` with the real notification polling endpoint
- [ ] End-to-end test: validate → submit → poll → download → upload to Manage Files
- [ ] Handle edge cases: dynamic component list (only present-in-course items), download URL expiration, very large ZIPs

---

## Phase 5: Advanced Capabilities

### 18. Dynamic Tool Management (Medium-High complexity)
**Prerequisite for**: Skills System (skills add tools, need dynamic loading to keep context lean), Agent Orchestration (agent tools benefit from dynamic activation)
**Depends on**: Nothing directly, but benefits from having the agent loop stable
**File**: `TODO-TOOL-MANAGEMENT.md`

Framework-level change. Must be backward compatible with existing agents.

- [ ] Define `coreTools` config option for agents
- [ ] `activate_tools` meta-tool (loads additional categories mid-conversation)
- [ ] Conversation-level tool state tracking (active categories across turns)
- [ ] Update `getToolDefinitionsForLLM` with `coreTools` + `activeCategories` options
- [ ] Auto-generate category descriptions for system prompt
- [ ] Dev UI updates (core tools visualization, token savings estimates)
- [ ] Backward compatibility: agents without `coreTools` config work unchanged

### 19. Skills System (High complexity)
**Depends on**: Schemas, Tool Management, Registry (for program-level storage)
**File**: `TODO-SKILLS-SYSTEM.md`

The largest body of work. Implement in sub-phases:

**19a. Framework Skills (Tier 1)**
- [ ] `frontend/src/skills/` directory convention
- [ ] Skill manifest schema (`index.json` per skill folder)
- [ ] `import.meta.glob` discovery of framework skills
- [ ] `data/skills.json` per-agent enablement config
- [ ] Skills tab in dev shell (toggle framework skills per agent)
- [ ] `load_skill` tool — resolves framework skill files via dynamic import

**19b. Program Skills (Tier 2)**
- [ ] Runtime discovery via `listCourseFolder` + `readCourseFile` for manifests
- [ ] Merge runtime manifests with framework manifests into unified catalog
- [ ] Admin UI for uploading/editing program skill packages
- [ ] `load_skill` tool — extended to resolve course file skills

**19c. Personal Skills (Tier 3)**
- [ ] Per-user skill storage (`{agent}/skills/user_{userId}/`)
- [ ] `save_skill` tool for LLM-driven personal skill creation
- [ ] User-facing settings panel for managing personal skills
- [ ] Skill editing (read markdown, present in textarea, write back)

**19d. Skill Catalog & UI**
- [ ] System prompt catalog generation (names + descriptions + applicability, not full content)
- [ ] Precedence: Personal > Program > Framework
- [ ] User-facing skills panel showing active skills grouped by tier
- [ ] Per-skill toggle (enable/disable without deleting)

### 20. Agent-as-Tool Orchestration (Medium-High complexity)
**Prerequisite for**: ID Assistant functioning as a true orchestrator across agents
**Depends on**: Tool Management (#18), multiple working agents (Course Review, Course Export, Template Fidelity)
**File**: `TODO-AGENT-ORCHESTRATION.md`

Enables a higher-level agent (e.g., the ID Assistant) to delegate tasks to specialized agents as LLM tool calls. Each agent declares headless `actions` in its manifest with typed input/output schemas. The framework auto-generates tool definitions and routes invocations through an `invoke()` handler.

**20a. Types & Manifest Extension**
- [ ] `AgentAction` interface and `InvokeContext` type in `shared/src/types/plugin.ts`
- [ ] Add optional `actions` array to `PluginManifest`
- [ ] Add optional `invoke` handler to `AgentPlugin` type
- [ ] Update `pluginManifestSchema` (Zod) to validate the `actions` field

**20b. Framework Invocation Layer**
- [ ] Create `core/plugins/invoker.ts` with `invokeAgentAction()` function
- [ ] Input validation against `action.inputSchema`
- [ ] Error handling (action not found, invoke failed, timeout)
- [ ] Invocation logging (start, duration, success/failure)

**20c. Tool Generation & Execution**
- [ ] Create `core/tools/agentTools.ts` — `buildAgentToolDefinitions()` from catalog
- [ ] Register `'agent'` as a new tool category
- [ ] Route agent tool calls in the tool executor to `invokeAgentAction()`
- [ ] Update `getToolDefinitionsForLLM()` to include agent tools when `'agent'` category is requested

**20d. Adopt in Existing Agents**
- [ ] Course Review: add `actions` to manifest, implement `invoke()` wrapping `executeWorkflow()`
- [ ] Course Export: add `actions` to manifest, implement `invoke()` wrapping `runExportBatch()`
- [ ] Template Fidelity: add `actions` to manifest, implement `invoke()`
- [ ] Course Embeddings: add `actions` to manifest for on-demand embedding runs

**20e. Integration & Observability**
- [ ] Agent catalog dev UI: show declared actions per agent
- [ ] Action invocation log in dev UI
- [ ] Handle long-running actions (synchronous first, async polling if needed)
- [ ] Handle `requiresUI` flag for UI-dependent actions (advisory handoff vs. headless)

---

## Phase 6: Polish & Scale

### 21. Journal Tier 3 — Semantic Search (Medium complexity)
**Depends on**: Journal Tier 1/2, course-embeddings pipeline
**File**: `TODO-DATA-JOURNAL.md` (Tier 3 section)

Only needed when the journal grows large enough that Tier 2 structured search is insufficient.

- [ ] Embed journal entries using the course-embeddings pipeline
- [ ] Store journal embeddings in Parquet alongside course content
- [ ] Integrate similarity search into `search_notes` tool
- [ ] Change detection: only re-embed new/modified entries

### 22. Skill Embedding & Discovery (Low-Medium complexity)
**Depends on**: Skills System, course-embeddings pipeline
**File**: `TODO-SKILLS-SYSTEM.md` (final section)

Only needed if the skills library grows beyond ~30 skills where keyword matching becomes unreliable.

- [ ] Embed skill descriptions and keywords
- [ ] Semantic search over skill manifests when deciding which to load
- [ ] Replace keyword-based catalog with embedding-based relevance ranking

---

## Dependency Graph

```
                           ┌──────────────────────────────────────────────────────┐
                           │ Phase 1–2: ID Assistant Foundations                  │
                           │                                                      │
                           │ Schemas ──┬──> Identity ──┐                          │
                           │           │               ├──> Registry ──> Journal  │
                           │ Data Load ┘               │                          │
                           └───────────────────────────┼──────────────────────────┘
                                                       │
                    ┌──────────────────────────────────┐│
                    │ Phase 3: Core Gaps (parallel)     ││
                    │                                   ││
                    │ Dropbox Submission ──────────┐    ││
                    │ Sibling-File Loading ────────┤    ││
                    │ Topic-Scoped Context ────────┤    ││
                    │ Transcript Tracking ─────────┤    ││
                    │ Voice Mode ──────────────────┤    ││
                    │ Reusable RAG ────────────────┤    ││
                    └──────────────────────────────┤────┘│
                                                   │     │
                    ┌──────────────────────────────┐│     │
                    │ Phase 4: Learning Agent       ││    │
                    │                               ││    │
                    │ Scaffolding ──> Outcome UI ───┤│    │
                    │       └──> Submission Flow ───┤│    │
                    │                       └──> Chat View │
                    │ RAG Integration ──────────────┤│    │
                    │ Deployment & Testing ─────────┘│    │
                    └────────────────────────────────┘    │
                                                          │
                    ┌─────────────────────────────────────┘
                    │ Phase 5–6: Advanced & Polish
                    │
                    │ Tool Mgmt ──> Skills System ──> Agent Orchestration
                    │ Journal T3                         ↑
                    │ Skill Embeddings       (also needs working agents)
                    └──────────────────────────────────────

     Independent Track (can proceed anytime):
     ─────────────────────────────────────────
     Course Export Wizard Completion ──> (feeds into Agent Orchestration #20d)
```

## Estimated Effort

| Phase | Items | Relative effort |
|-------|-------|----------------|
| Phase 1: Foundations | Schemas, Identity, Data Loading | Small-Medium |
| Phase 2: Hub Core | Registry, Journal T1/T2 | Medium |
| Phase 3: Core Gaps | Dropbox, Sibling-File, Topic Context, Transcript, Voice, RAG | Medium-Large |
| Phase 4: Learning Agent | Scaffolding, Outcomes, Submission, Chat, RAG integration | Medium |
| Phase 5: Advanced | Tool Management, Skills System, Agent Orchestration | Large |
| Phase 6: Polish | Journal T3, Skill Embeddings | Small (incremental) |
| Independent | Course Export Wizard Completion | Medium (empirical / investigation-heavy) |

**Parallelism notes**:
- Phase 1–2 (ID Assistant foundations) and Phase 3 (core gaps) can be developed **in parallel** — they have no mutual dependencies.
- Within Phase 3, all six core gaps are independent of each other and can be tackled in any order or concurrently.
- Phase 4 (Learning Agent) depends on Phase 3 completions but its agent-level scaffolding (config parser, outcome UI, prompt builder) can start before all core gaps are finished.
- Phase 5–6 are independent of the Learning Agent and can proceed whenever Phase 1–2 is stable.
- The **Course Export Wizard** completion is an independent track — it requires live Brightspace access for DevTools investigation and can be done anytime. It feeds into Agent Orchestration (#20d) since the export agent must work before it can be delegated to.
- **Agent Orchestration** (#20) is placed last in Phase 5 because it benefits from having Tool Management in place and multiple agents fully functional. Its per-agent adoption (#20d) can be done incrementally as agents are completed.
