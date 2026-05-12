# TODO: Agent-as-Tool Orchestration

## Context

As the agent ecosystem grows (Course Review, Course Export, Template Fidelity, Learning Agent, Course Embeddings, etc.), the opportunity arises for a higher-level **orchestrator agent** — such as an Instructional Design Assistant — that can delegate tasks to specialized agents based on user intent.

Rather than hardcoding cross-agent imports, this TODO describes a framework-level pattern where agents **declare headless actions** in their manifests and an orchestrator agent's LLM **invokes them as tools** during conversation.

## What Exists Today

- **Agent catalog** (`agent-manager/catalog.ts`) — discovers all agents at build time via `import.meta.glob`, loads manifests, and enumerates capabilities.
- **Plugin manifest** (`PluginManifest`) — declares `id`, `name`, `version`, `description`, `capabilities`, `requiredInputs`.
- **Workflow engine** (`core/workflow/engine.ts`) — `executeWorkflow()` runs a workflow definition headlessly given input + execution context (`ou`, `xsrfToken`, `coursePath`).
- **Tool system** — `ToolDefinition`, `executeBrightspaceTool()`, `getToolDefinitionsForLLM()`, category-based filtering, allowlists.
- **Dynamic tool management** (planned, `TODO-TOOL-MANAGEMENT.md`) — `activate_tools` meta-tool, core tool sets, on-demand category activation.

### Current agent types

| Pattern | Examples | Headless today? |
|---------|----------|----------------|
| LLM-workflow | Course Review, Template Fidelity | Yes — `executeWorkflow()` runs without UI |
| Service-driven (no LLM) | Course Export | Partially — orchestrator service works headlessly, but wizard automation is tightly coupled to the browser session |
| Pipeline | Course Embeddings | Yes — runs as a batch process |

## Proposed Approach: Agent-as-Tool

Each agent optionally declares one or more **actions** — headless entry points with typed inputs and outputs. The framework auto-generates LLM tool definitions from these action declarations. When an orchestrating agent's LLM calls the tool, the framework resolves it to the target agent's `invoke()` handler.

### How it works

```
User: "Can you run a QM review on EDIT 7350?"
            │
            ▼
   ID Assistant LLM
   (sees tool: run_course_review)
            │
            ▼
   LLM calls: run_course_review({ ou: "12345", standard: "qm" })
            │
            ▼
   Framework: invokeAgentAction("course-review", "review", { ou: "12345", standard: "qm" }, context)
            │
            ▼
   Course Review agent's invoke() handler
   → executeWorkflow(courseReviewWorkflow, ...) OR custom logic
            │
            ▼
   Result returned as tool response to ID Assistant LLM
```

The LLM decides *when* to delegate based on conversation context. The orchestrator agent doesn't need to know implementation details of the sub-agent — it just sees the tool definition.

---

## Design

### 1. Manifest Extension: `actions`

Add an optional `actions` array to `PluginManifest`:

```typescript
interface AgentAction {
  /** Unique action ID within this agent */
  id: string;

  /** Human-readable description (becomes the tool's description) */
  description: string;

  /**
   * JSON Schema for the action's input parameters.
   * Becomes the tool's `parameters` schema.
   */
  inputSchema: Record<string, unknown>;

  /**
   * Description of what the action returns.
   * Included in the tool description to help the LLM understand the output.
   */
  outputDescription?: string;

  /**
   * Whether this action requires a UI to function properly.
   * If true, the orchestrator may choose to hand off to the
   * specialized UI rather than running headlessly.
   */
  requiresUI?: boolean;

  /**
   * Estimated execution time. Helps the orchestrator set expectations
   * with the user ("this will take a few minutes").
   */
  estimatedDuration?: 'seconds' | 'minutes' | 'long-running';

  /**
   * Tool categories this action needs internally.
   * Used by the framework to ensure the right tools are available
   * when the action executes.
   */
  internalToolCategories?: string[];
}
```

Example manifest for Course Review:

```typescript
export const agentPlugin = defineAgentPlugin({
  manifest: {
    id: 'course-review',
    name: 'CourseReview',
    version: '1.0.0',
    description: 'AI-powered course review against QM and OSCQR rubrics',
    pluginApiVersion: '1.0',
    frameworkApiVersion: '1.0',
    requiredInputs: requiredInputs,
    capabilities: ['brightspace-tools', 'state-persistence'],
    actions: [
      {
        id: 'review',
        description: 'Run an automated course review against a quality standard. Returns a structured review with findings per standard item.',
        inputSchema: {
          type: 'object',
          properties: {
            ou: { type: 'string', description: 'Org unit ID of the course to review' },
            standard: { type: 'string', enum: ['qm', 'oscqr'], description: 'Quality standard to review against' },
          },
          required: ['ou', 'standard'],
        },
        outputDescription: 'JSON object with review findings, scores, and recommendations per standard item',
        estimatedDuration: 'minutes',
        internalToolCategories: ['content', 'course', 'discussions', 'grades', 'quizzes', 'dropbox', 'rubrics'],
      },
    ],
  },
  workflow: courseReviewWorkflow,
});
```

Example manifest for Course Export:

```typescript
export const agentPlugin = defineAgentPlugin({
  manifest: {
    id: 'course-export',
    name: 'CourseExport',
    version: '1.0.0',
    description: 'Batch export Brightspace courses as ZIP packages',
    pluginApiVersion: '1.0',
    frameworkApiVersion: '1.0',
    actions: [
      {
        id: 'export',
        description: 'Export one or more Brightspace courses as ZIP packages and upload them to the host course Manage Files area.',
        inputSchema: {
          type: 'object',
          properties: {
            ouIds: {
              type: 'array',
              items: { type: 'string' },
              description: 'Org unit IDs of courses to export',
            },
            includeCourseFiles: {
              type: 'boolean',
              description: 'Whether to include course files in the export package',
              default: true,
            },
          },
          required: ['ouIds'],
        },
        outputDescription: 'Summary of export results: per-course status, file names, sizes, and any errors',
        estimatedDuration: 'long-running',
      },
    ],
  },
});
```

### 2. The `invoke()` Handler

Each agent that declares actions provides an `invoke()` function on its plugin:

```typescript
export const agentPlugin = defineAgentPlugin({
  manifest: { /* ... actions declared here ... */ },
  workflow: courseReviewWorkflow,

  invoke: async (actionId, input, context) => {
    switch (actionId) {
      case 'review':
        return await runCourseReview(input.ou, input.standard, context);
      default:
        throw new Error(`Unknown action: ${actionId}`);
    }
  },
});
```

The `invoke()` function receives:

```typescript
interface InvokeContext {
  /** The org unit of the orchestrating agent (host course) */
  ou: string;
  xsrfToken: string | null;
  coursePath: string | null;
  /** The user's identity from whoAmI */
  userId: string;
}

type InvokeHandler = (
  actionId: string,
  input: Record<string, unknown>,
  context: InvokeContext,
) => Promise<unknown>;
```

For LLM-workflow agents, the `invoke()` handler can simply call `executeWorkflow()`:

```typescript
async function runCourseReview(ou: string, standard: string, context: InvokeContext) {
  const result = await executeWorkflow(
    courseReviewWorkflow,
    { standardSetId: standard },
    { ou, xsrfToken: context.xsrfToken, coursePath: context.coursePath },
  );
  return result.output;
}
```

For service-driven agents like Course Export, the handler calls the service layer directly:

```typescript
async function runCourseExport(ouIds: string[], includeCourseFiles: boolean, context: InvokeContext) {
  const courses = await validateCourses(ouIds);
  const valid = courses.filter(c => c.isValid);
  if (valid.length === 0) {
    return { success: false, error: 'No valid courses found', validation: courses };
  }

  const jobs = valid.map((course, i) => ({
    id: `export-${course.ouId}-${Date.now()}-${i}`,
    course,
    status: 'queued' as const,
    retryCount: 0,
  }));

  const logs: LogEntry[] = [];
  await runExportBatch(jobs, EXPORT_TOOLS, context.ou, context.xsrfToken!, {
    onJobUpdate: () => {},
    onLog: (entry) => logs.push(entry),
  }, { includeCourseFiles });

  return {
    success: true,
    results: jobs.map(j => ({
      ou: j.course.ouId,
      name: j.course.name,
      status: j.status,
      fileName: j.fileName,
      fileSize: j.fileSize,
      error: j.error,
    })),
    logs,
  };
}
```

### 3. Framework: `invokeAgentAction()`

A new framework function that resolves an agent by ID, validates the action, and calls its `invoke()` handler:

```typescript
// core/plugins/invoker.ts

async function invokeAgentAction(
  agentId: string,
  actionId: string,
  input: Record<string, unknown>,
  context: InvokeContext,
): Promise<unknown> {
  const catalog = await loadAgentCatalog();
  const entry = catalog.find(e => e.id === agentId);
  if (!entry) throw new Error(`Agent "${agentId}" not found in catalog`);

  const action = entry.manifest?.actions?.find(a => a.id === actionId);
  if (!action) throw new Error(`Agent "${agentId}" has no action "${actionId}"`);

  // Dynamically load the agent module and call invoke()
  const module = await loadAgentModule(agentId);
  if (!module.agentPlugin?.invoke) {
    throw new Error(`Agent "${agentId}" declares action "${actionId}" but has no invoke() handler`);
  }

  // TODO: Validate input against action.inputSchema (via Zod or ajv)

  return module.agentPlugin.invoke(actionId, input, context);
}
```

### 4. Auto-Generated Tool Definitions

The framework generates LLM tool definitions from agent action declarations:

```typescript
// core/tools/agentTools.ts

function buildAgentToolDefinitions(catalog: AgentCatalogEntry[]): ToolDefinition[] {
  const tools: ToolDefinition[] = [];

  for (const agent of catalog) {
    if (!agent.manifest?.actions) continue;

    for (const action of agent.manifest.actions) {
      tools.push({
        name: `${agent.id}__${action.id}`,    // e.g. "course-review__review"
        description: `[${agent.name}] ${action.description}`,
        parameters: action.inputSchema,
        category: 'agent',                      // new tool category
      });
    }
  }

  return tools;
}
```

The tool executor routes `agent`-category tools to `invokeAgentAction()`:

```typescript
// In the tool execution pipeline
if (toolName.includes('__') && category === 'agent') {
  const [agentId, actionId] = toolName.split('__');
  const result = await invokeAgentAction(agentId, actionId, args, context);
  return JSON.stringify(result);
}
```

### 5. Orchestrator Agent Configuration

The ID Assistant opts in to agent tools like any other tool category:

```typescript
const idAssistantWorkflow = workflow('id-assistant')
  .ai('main', {
    tools: [
      'content', 'course',       // standard Brightspace tool categories
      'agent',                    // all agent actions become available as tools
    ],
    maxTurns: 30,
  })
  .build();
```

Or with dynamic tool management (`activate_tools`), agent tools could be activated on demand — the system prompt would list available agents and the LLM activates the ones it needs.

---

## Handling Long-Running Actions

Some agent actions (Course Export, Course Review) take minutes to complete. The synchronous `invoke()` → tool response flow works fine for short actions, but for long-running ones the LLM call would hang.

### Options

**Option A: Synchronous with LLM timeout extension**

The simplest approach. The workflow engine already supports long-running AI steps. The tool call just takes longer. The orchestrating LLM waits. This works if the LLM provider doesn't time out (most allow several minutes for tool calls).

Downside: the user sees no progress during the wait.

**Option B: Async with status polling**

```typescript
// invoke() returns immediately with a job handle
const handle = await invokeAgentAction('course-export', 'export', input, context);
// => { jobId: "abc-123", status: "running" }

// LLM checks status in subsequent turns
const status = await checkAgentJob('abc-123');
// => { status: "complete", result: { ... } }
```

This requires a new `check_agent_job` tool and an in-memory job registry. More complex but gives the orchestrator (and user) visibility into progress.

**Option C: Streaming progress via callbacks**

The `invoke()` handler could accept a `onProgress` callback that feeds status updates back to the orchestrating agent's UI (not the LLM, but a status panel).

### Recommendation

Start with **Option A** for simplicity. LLM-workflow agents (Course Review) already run in `executeWorkflow()` which handles multi-turn loops internally. Service-driven agents (Course Export) run synchronously through their orchestrator. If timeouts become a problem, add Option B.

---

## The `requiresUI` Flag

Some actions may not work well headlessly. For example, the Course Export agent's wizard automation requires an active browser session, but the *progress monitoring* and *retry* UX is much better in the specialized UI.

When an action has `requiresUI: true`, the orchestrating agent could respond to the user differently:

> "I can start a course export for you. Since this involves monitoring progress and handling retries, I'll open the Course Export tool with your courses pre-configured. Would you like me to proceed?"

The orchestrator could:
1. Render the sub-agent's `MainView` component in a panel/modal within its own UI
2. Navigate the user to the sub-agent's standalone page with pre-filled parameters (via URL query params or shared state)
3. Run it headlessly anyway but with reduced UX fidelity

The `requiresUI` flag is advisory — the orchestrator decides how to handle it based on its own design.

---

## Relationship to Other Systems

### Dynamic Tool Management (`TODO-TOOL-MANAGEMENT.md`)

Agent tools are a new **tool category** (`'agent'`). They integrate naturally with the dynamic tool management system:

- With `activate_tools`: agent tools could be activated on demand like any other category
- With `coreTools`: an orchestrator could make specific agent actions always-present
- Category descriptions would auto-generate from action declarations

### Skills System (`TODO-SKILLS-SYSTEM.md`)

Skills and agent actions are complementary:
- **Skills** = knowledge and procedures the LLM follows (markdown instructions, examples)
- **Agent actions** = executable capabilities the LLM delegates to

A skill could *reference* an agent action: "When the user asks for a course review, use the `course-review__review` tool." But the skill itself is still declarative — it tells the LLM *when* to delegate, not *how*.

### Agent Catalog (`agent-manager/catalog.ts`)

The catalog already discovers agents and loads manifests. The `actions` field extends what's already there. The auto-generation of tool definitions plugs into `loadAgentCatalog()` naturally.

---

## Work Items

### Phase A: Manifest & Types

- [ ] Add `AgentAction` interface to `shared/src/types/plugin.ts`
- [ ] Add optional `actions` array to `PluginManifest`
- [ ] Add optional `invoke` handler to `AgentPlugin` type
- [ ] Update `pluginManifestSchema` (Zod) to validate the `actions` field
- [ ] Update `InvokeContext` type with `ou`, `xsrfToken`, `coursePath`, `userId`
- [ ] Export all new types from `@uga-brightspace/shared`

### Phase B: Framework Invocation Layer

- [ ] Create `core/plugins/invoker.ts` with `invokeAgentAction()` function
- [ ] Add input validation against `action.inputSchema` (Zod or ajv)
- [ ] Handle errors uniformly (action not found, invoke failed, timeout)
- [ ] Logging: log invocation start, duration, success/failure
- [ ] Consider: should invocations be sandboxed? (e.g., the sub-agent shouldn't be able to write to the orchestrator's state folder)

### Phase C: Tool Generation & Execution

- [ ] Create `core/tools/agentTools.ts` — `buildAgentToolDefinitions()` from catalog
- [ ] Register `'agent'` as a new tool category
- [ ] Route agent tool calls in the tool executor to `invokeAgentAction()`
- [ ] Update `getToolDefinitionsForLLM()` to include agent tools when the `'agent'` category is requested
- [ ] Tool naming convention: `{agentId}__{actionId}` (double underscore to avoid conflicts with existing tool names)

### Phase D: Adopt in Existing Agents

- [ ] Course Review: add `actions` to manifest, implement `invoke()` wrapping `executeWorkflow()`
- [ ] Course Export: add `actions` to manifest, implement `invoke()` wrapping `runExportBatch()` (headless mode)
- [ ] Template Fidelity: add `actions` to manifest, implement `invoke()`
- [ ] Course Embeddings: add `actions` to manifest, implement `invoke()` for on-demand embedding runs

### Phase E: Orchestrator Agent (ID Assistant)

- [ ] Include `'agent'` in the ID Assistant's tool categories
- [ ] System prompt guidance: when to delegate vs. handle directly
- [ ] Handle long-running action responses (user messaging, progress)
- [ ] Handle `requiresUI` actions (decide on handoff strategy)
- [ ] Test: multi-step delegation (e.g., "review this course then export it")

### Phase F: Dev UI & Observability

- [ ] Agent catalog view: show declared actions per agent
- [ ] Action invocation log: track which actions were invoked, by whom, duration, result
- [ ] Dev-mode action testing: invoke an action manually from the agent manager UI

---

## Dependency Position

This feature sits between the **Dynamic Tool Management** work and the **ID Assistant** agent build. It could be implemented:

- **After** Tool Management (Phase 5, item 18) — agent tools benefit from dynamic activation
- **Before** the ID Assistant's conversational workflow is built — the assistant needs agent tools to be useful as an orchestrator

Suggested placement in `TODO-IMPLEMENTATION-ORDER.md`:

```
Phase 5: Advanced Capabilities
  18. Dynamic Tool Management
  18b. Agent-as-Tool Orchestration    ← new
  19. Skills System
```

### Complexity: Medium

- Types & manifest extension: Low
- Invocation layer: Low-Medium (mostly plumbing)
- Tool generation & routing: Low (follows existing patterns)
- Per-agent adoption: Low per agent (small `invoke()` handler)
- Orchestrator integration: Medium (system prompt design, UX decisions for long-running / UI-required actions)

---

## Open Questions

- [ ] Should agent tools be namespaced as `{agentId}__{actionId}` or use a flatter naming like `run_{agentId}_{actionId}`? The double-underscore convention is unambiguous but less readable in LLM tool calls.
- [ ] Should the orchestrator be able to pass its own conversation history or relevant context to the sub-agent's `invoke()`, or is the typed `input` sufficient?
- [ ] Should `invoke()` be allowed to call *other* agent actions (recursive delegation)? If so, what's the depth limit?
- [ ] Should action results be cached? (e.g., "you already reviewed this course 10 minutes ago, here are the results")
- [ ] Should the framework enforce that agents declaring `actions` must also export an `invoke()` handler, or fail gracefully at runtime?
- [ ] For long-running actions: should there be a framework-level timeout, or is it purely the orchestrator's responsibility?
