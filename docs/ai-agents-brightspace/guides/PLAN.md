# Brightspace Agent Framework — Planning Prompt

## Your Role

You are planning the architecture and implementation of a reusable framework for building AI-powered agent applications that run inside the Brightspace LMS. This framework will be used by a small team at the University of Georgia's online learning division. Your job is to produce a detailed implementation plan before any code is written.

## Context & Vision

We have built several AI agent apps that run inside Brightspace as embedded React applications. These are **not chatbots** — they are task-driven, workflow-oriented agents that use AI as an engine to power structured educational processes. Chat is an optional secondary feature, not the core interaction model.

### Existing agents built on this pattern:

- **Course Starter App**: Ingests an uploaded syllabus, uses AI to generate foundational course materials, and writes them directly to Brightspace as content objects.
- **Course Review App**: Loads review rubrics (e.g., Quality Matters) from JSON files, then uses tool calls to iterate through each criterion and conduct a structured review of course content.
- **Module Builder App**: Reads the framework/outline created by Course Starter and continues building detailed content for each module in the course.

### Key characteristics of these agents:

- They are **workflow/pipeline-driven**, not conversation-driven.
- They consume structured inputs: markdown prompt files, JSON schemas, uploaded documents, and LMS context data.
- They produce structured outputs: course content objects written directly to Brightspace via the Valence API.
- They persist state as JSON or markdown files in Brightspace course files, allowing agents to pause and resume across sessions.
- Agents can depend on each other's output (Module Builder consumes Course Starter's framework).
- Workflows are **not always linear** — they involve iteration (looping over rubric criteria or modules), branching (conditional paths based on AI output or user decisions), user gates (pause for approval before proceeding), and potentially parallel execution.

### The goal:

Build a framework that extracts and standardizes the common patterns from these agents into a reusable foundation. A developer (or an AI coding assistant like Cursor) should be able to pull down this framework, describe what a new agent should do, and build the custom functionality on top of the framework's standard infrastructure.

## Reference Codebase

The existing Course Review app contains battle-tested implementations of many patterns this framework needs to generalize. The source is located at:

```
/home/jcastle/local_dev/uga-brightspace-course-review/frontend
```

Before planning, **read and analyze the following from that codebase** to understand existing patterns:

1. **LLM Service** (`llmService.ts` or similar): How AI API calls are made, tool call handling, streaming, provider support.
2. **Brightspace API services**: How Valence API calls are made for reading course data and writing content objects. Look for patterns around module creation, topic creation, file handling, and content publishing.
3. **Context management**: How LMS context (org unit, XSRF token, course path, user role) is loaded and made available throughout the app.
4. **File handling**: How the app reads from and writes to Brightspace course files (JSON state, markdown prompts).
5. **Tool definitions**: How tools/function calls are defined, their schemas, and how tool results are processed.
6. **State management**: How Zustand stores are structured and how workflow state is managed.
7. **Any workflow/pipeline patterns**: How the app sequences multi-step AI operations (e.g., iterating over rubric criteria).

**Extract and generalize these patterns rather than building from scratch.** The framework should feel like a natural evolution of this codebase, not a rewrite.

## Tech Stack

The framework uses the following technologies. Do not introduce alternatives unless there is a compelling reason.

| Concern | Technology |
|---------|-----------|
| Framework & Language | React 19, TypeScript 5.9 |
| Build Tool | Vite 6 with `@vitejs/plugin-react-swc` |
| Testing | Vitest |
| State Management | Zustand 5 (global state), React useState (local UI) |
| AI API Calls | Direct `fetch()` via custom LLM service — no Vercel AI SDK |
| Brightspace API Calls | Axios |
| LLM Providers | OpenAI, OpenRouter, Airia (all OpenAI-compatible API shape) |

### Authentication

The app runs inside Brightspace and uses the authenticated user's session. **There is no separate auth system.** The app inherits the user's Brightspace session, XSRF token, and permissions.

### Deployment

The React app is built and the packaged JS is uploaded directly to Brightspace, where it executes natively in the browser within the LMS context.

## Repository Structure

The framework is a monorepo:

```
uga-online-brightspace-agent-framework/
├── backend/        ← Thin API proxy (new)
├── frontend/       ← React framework + agent scaffold
└── shared/         ← TypeScript types, interfaces, schemas shared between frontend and backend
```

## Architecture: Core Systems

Plan each of these systems in detail. For each, specify the directory structure, key files/modules, TypeScript interfaces, and how they interact with other systems.

---

### 1. Workflow Engine (Core)

This is the heart of the framework. It orchestrates multi-step AI-powered processes.

**Requirements:**

- Support these workflow patterns:
  - **Sequential**: Step A → Step B → Step C
  - **Iteration/Looping**: Execute a step for each item in a collection (e.g., each rubric criterion, each module)
  - **Branching/Conditional**: Take different paths based on AI output, data state, or user decisions
  - **User Gates**: Pause workflow, present output to user, wait for approval/direction before continuing
  - **Parallel**: Execute independent steps concurrently (design for this even if not implemented in v1)

- **State persistence**: Workflows serialize their state to Brightspace course files (JSON). A user can close the browser and resume later. The engine checks for existing state on startup and resumes from the last completed step.

- **Step definition**: Each step specifies:
  - What it does (AI call, tool execution, data transformation, user gate)
  - Its required inputs (from previous steps, from context, from Brightspace)
  - Its output type
  - Its prompt template (if it involves an AI call)
  - Its tools (if applicable)

- **Prefer a programmatic/composable API** for defining workflows (builder pattern or composable functions), not a purely declarative config format. This gives maximum flexibility and is easier for AI coding assistants to work with. Example conceptual shape (not prescriptive — design the actual API):

```typescript
workflow('course-review')
  .input(rubricSchema, courseContent)
  .step('loadRubric', loadRubricTool)
  .forEach('criteria', 'rubric.criteria')
    .step('evaluate', evaluateCriterionPrompt)
    .gate('userReview', { display: 'rubricResult' })
  .end()
  .step('aggregate', aggregateResultsPrompt)
  .output(reviewReport)
```

---

### 2. Tool System

A standardized system for defining, registering, validating, and executing tools (function calls) that AI models can invoke.

**Requirements:**

- **Tool registry**: Agents declare available tools. The framework manages registration and lookup.
- **Standard tool interface**: Each tool has an input schema (use Zod for validation and type inference), an execution function, an output schema, and error handling.
- **Built-in LMS tools**: The framework ships with standard tools for common Brightspace operations (fetch content, read rubric, get grade items, etc.). Agent developers add custom tools on top.
- **Tool middleware**: Pre/post hooks for logging, transformation, or gating tool execution.
- **Schema editing**: Support for editing tool schemas and testing them locally during development. (The existing codebase has a schema editor — review it and determine what to generalize.)

---

### 3. Brightspace Integration Layer

A standardized abstraction over the Brightspace Valence API. This replaces the per-agent API implementations with a shared, typed client.

**Three sub-layers:**

#### 3a. Read Layer
Fetching course context and content:
- Course info (org unit, name, structure)
- User info and role
- Content objects (modules, topics, files, HTML)
- Rubrics, grade items, assignments
- Discussion forums/posts
- Calendar events

#### 3b. State Layer
Reading and writing workflow state files in Brightspace course files:
- Standard envelope format wrapping agent-specific data with workflow metadata (current step, completed steps, iteration index, timestamps)
- Predictable file locations per agent instance
- Save on step completion and user gates; load and resume on startup

#### 3c. Content Layer
Creating and managing course structure objects:
- Module creation
- Topic creation (HTML, file, link)
- Content publishing
- **Content staging**: Generate into intermediate representation first, then publish as a separate step with user approval
- **Idempotency and error recovery**: Track what was created so partial failures can be resumed, not duplicated

**Role-based capability gating**: The integration layer should be aware of user roles (instructor, student, TA, designer) and expose or restrict capabilities accordingly.

**Reference**: Extract and generalize the Brightspace API patterns from the existing Course Review app.

---

### 4. Prompt Management System

A structured system for managing the prompts that drive each workflow step.

**Requirements:**

- **Per-step prompt templates**: Each workflow step that involves an AI call has its own prompt template, stored as a markdown file.
- **Variable interpolation**: Templates support variables that get resolved at runtime (e.g., `{{rubric_content}}`, `{{course_objectives}}`, `{{module_outline}}`). A simple interpolation engine — Handlebars-level complexity is fine but a lightweight custom solution is also acceptable.
- **Layered prompt composition**:
  - System-level defaults (safety, institutional policies, output format instructions)
  - Agent-level instructions (the agent's purpose and persona)
  - Step-level prompt (the specific task for this workflow step)
  - Dynamic context (injected data from Brightspace, previous steps, uploaded files)
- **Local editing**: Prompts can be edited as markdown files in the project, with changes reflected immediately during development (leverage Vite HMR).

---

### 5. Context Ingestion System

A standardized way for agents to declare, load, validate, and inject their required inputs.

**Input types to support:**

- Markdown files (prompt templates, instructional content)
- JSON files (rubric schemas, configuration, framework definitions)
- Uploaded documents (syllabi, PDFs, other user-provided files)
- LMS data (course content, rubrics, enrollments — fetched via the Brightspace integration layer)
- Output from other agents (e.g., Module Builder consuming Course Starter's framework file from a known location in course files)

**Requirements:**

- Agents declare required inputs and their types in the workflow definition
- The framework validates that all required inputs are available before starting a workflow
- Inputs are loaded and made available to workflow steps through a typed context object
- Inter-agent dependencies are resolved through file contracts: agents produce output files in predictable locations; dependent agents declare those files as inputs

---

### 6. Structured Output System

Handles the outputs produced by workflow steps and the final deliverables.

**Two output modes:**

- **Internal outputs**: JSON/markdown state files written to Brightspace course files. Used for workflow persistence and inter-agent communication.
- **LMS content outputs**: Structured objects created directly in the Brightspace course (modules, topics, HTML content). These are the deliverables that instructors and students interact with.

**Requirements:**

- Typed output schemas for each workflow step
- Outputs from one step can be piped as inputs to subsequent steps
- **Content staging**: LMS content outputs go through an intermediate representation before publishing. The user reviews and approves before content is written to the course.
- **Rollback awareness**: Track what content objects have been created so that partial failures can be identified and either cleaned up or resumed.

---

### 7. Backend Proxy

A thin backend service that handles API key management and proxies AI requests.

**Purpose**: Currently, API keys are entered by users and stored in localStorage. The backend proxy eliminates this by holding keys server-side.

**Requirements:**

- Receives AI API requests from the frontend React apps
- Validates the request originates from a legitimate Brightspace session
- Attaches the appropriate API key
- Forwards to the configured AI provider
- Streams the response back to the frontend
- Simple configuration for routing to different providers/models per agent
- Optional: basic usage logging and rate limiting

**Keep this minimal.** No database, no user management, no business logic beyond proxying. Brightspace handles auth and identity.

---

### 8. UI Component Library

Reusable React components for agent interfaces. **Chat is not the primary interaction model.**

**Core components needed:**

- **Workflow progress**: Visual indicator of which stage the agent is in, what's completed, what's pending
- **Structured input forms**: Upload areas, configuration panels, input selection (e.g., select rubric, upload syllabus)
- **Structured output displays**: Purpose-built views for agent outputs (review results rendered as scored rubrics, generated course materials in editable/previewable form, module outlines)
- **User gate UI**: Approval/rejection/redirect interface for decision points in workflows
- **Content staging preview**: Show what will be created in Brightspace before publishing
- **Chat module (optional)**: A drop-in chat sidebar that can be attached to any agent for Q&A or modification requests. Not required for every agent.

**Styling**: Use a consistent approach that can be themed to match institutional branding. Prefer a utility-first approach (Tailwind or similar) with shadcn/ui or Radix for primitives.

---

### 9. Agent Configuration & Scaffolding

**Agent definition**: Each agent built on the framework follows a standard structure:

```
agents/course-review/
├── workflow.ts          ← Workflow definition using the engine's API
├── tools/               ← Custom tools for this agent
├── prompts/             ← Markdown prompt templates for each step
├── components/          ← Custom UI components (if defaults don't fit)
├── content/             ← Content publishing logic
└── config.ts            ← Agent metadata, required inputs, output types
```

**Scaffolding**: A command or script that generates this structure for a new agent with boilerplate wired up:

```bash
npm run create-agent -- --name "accessibility-review"
```

**Example agents**: The framework ships with 2-3 complete example agents that demonstrate the framework's patterns. These serve as documentation and as few-shot examples for AI coding assistants building new agents.

---

## Design Priorities

1. **AI-legible**: This framework will primarily be extended by AI coding assistants (Cursor, Claude Code). Prioritize clear naming, strong TypeScript types, comprehensive inline documentation, and an `ARCHITECTURE.md` that explains the mental model. Include example agents as reference implementations.

2. **Convention over configuration**: Predictable directory structures, naming patterns, and extension points so that an AI assistant can scaffold new agents reliably.

3. **Strong typing**: TypeScript interfaces for everything — workflow steps, tool contracts, Brightspace API responses, content schemas, state envelopes. Use Zod for runtime validation with type inference.

4. **Extract, don't rewrite**: The existing Course Review app has working, tested implementations. Extract and generalize those patterns. The framework should feel like a natural evolution, not a from-scratch rebuild.

5. **Progressive complexity**: Simple agents should be simple to build. Complex agents should be possible. Don't force every agent through unnecessary abstraction.

---

## Planning Deliverables

Produce a detailed implementation plan that includes:

1. **Architecture diagram**: How the core systems connect and interact.
2. **Directory structure**: Complete file/folder layout for the monorepo, the framework itself, and the example agent structure.
3. **Key interfaces and types**: TypeScript interfaces for the core abstractions (WorkflowStep, Tool, BrightspaceClient, AgentConfig, WorkflowState, etc.). These don't need full implementation yet, but they should be concrete enough to code against.
4. **Implementation phases**: Break the build into ordered phases. What gets built first? What depends on what? Suggest a sequence that delivers usable functionality incrementally.
5. **Migration path**: How do we extract from the existing Course Review app into the framework? Which pieces come over as-is, which need generalization, and which are agent-specific and stay behind?
6. **Decisions and tradeoffs**: Flag any architectural decisions that have significant tradeoffs, present the options, and recommend an approach with reasoning.
7. **Example agent walkthrough**: Show how one of the existing agents (Course Review is a good candidate) would look when rebuilt on the framework. Walk through the workflow definition, tool setup, prompt templates, and content publishing.

---

## Important Notes

- Do NOT begin implementation until the plan is reviewed and approved.
- Do NOT introduce new dependencies or technologies without justifying why the existing stack is insufficient.
- Do NOT over-engineer. This framework serves a small team. Prefer simplicity and clarity over premature abstraction.
- When in doubt about existing patterns, **read the reference codebase first**.