# Brightspace Agent Framework — Coding Agent Guide

> **This document is written for AI coding agents.** If you are a human developer, read [DEVELOPER-GUIDE.md](DEVELOPER-GUIDE.md) instead.

Read this file before writing any code. It covers everything you need to implement a new agent plugin in this repository with no additional context.

---

## 1. What This Repository Is

This repo is a **framework only** — it does not include finished agent implementations. Agents are self-contained plugins that live in `frontend/src/agents/<agent-name>/`. You are building a new agent inside that directory.

The framework provides:
- A workflow builder and execution engine
- Brightspace Valence API tools (content, grades, discussions, etc.)
- An LLM client with multi-provider support
- State persistence via Brightspace course files
- A React UI shell and UGA design system integration

---

## 2. The Only Safe Import Path

Agent code must import all framework functionality from:

```typescript
import { ... } from '@uga-brightspace/framework';
```

**Never** import from `frontend/src/core/*` or any deep path. The lint guard (`scripts/check-agent-imports.js`) runs on every build and will reject the build if a violation is found.

The full export surface is defined in [`frontend/src/agent-api/index.ts`](frontend/src/agent-api/index.ts).

---

## 3. Agent File Structure

The canonical reference agent is `template-fidelity`. Use it as your primary example.

```
frontend/src/agents/my-agent/
├── config.ts             # workflow definition + agentPlugin manifest (REQUIRED)
├── entry.tsx             # React root — copy from template-fidelity, change agent name only
├── entry.html            # HTML loader — copy from template-fidelity, change title only
├── types.ts              # TypeScript types for this agent's data shapes
├── index.ts              # Public exports: agentPlugin, agentConfig, MainView
├── data/
│   ├── tools.json        # { "disabledTools": [] } — controls which Brightspace tools the LLM sees
│   └── schema.json       # Optional agent-specific JSON schema
├── prompts/              # Markdown prompt templates ({{variable}} interpolation syntax)
│   └── main.md           # Primary LLM system prompt
├── components/           # React components
│   └── MainView.tsx      # Main UI component (REQUIRED — entry.tsx renders this)
├── services/             # Business logic, LLM loops, Brightspace data fetching
└── tools/                # Custom LLM tool definitions (submit-output pattern)
```

---

## 4. config.ts Contract

Every agent must export three things from `config.ts`:

```typescript
import { defineAgentPlugin, workflow } from '@uga-brightspace/framework';
import type { PluginInputDeclaration } from '@uga-brightspace/framework';
import { z } from 'zod';

// 1. Workflow definition (used by the framework, not directly by agent logic)
export const myAgentWorkflow = workflow('my-agent')
  .name('MyAgent')
  .version('1.0')
  .input(z.object({}))
  .ai('main', {
    promptTemplate: 'Complete the task.',
    tools: ['content', 'course'],
    outputKey: 'result',
    maxTurns: 15,
  })
  .build();

// 2. Agent config (used for state folder naming)
export const agentConfig = {
  id: 'my-agent',
  name: 'MyAgent',
  version: '1.0.0',
  description: 'What this agent does',
  workflow: myAgentWorkflow,
  stateFolder: 'my-agent',
  stateFile: 'state.json',
};

// 3. Plugin manifest (required for compatibility checks)
export const agentPlugin = defineAgentPlugin({
  manifest: {
    id: 'my-agent',
    name: 'MyAgent',
    version: '1.0.0',
    description: 'What this agent does',
    pluginApiVersion: '1.0',
    frameworkApiVersion: '1.0',
    requiredInputs,            // optional
    capabilities: ['brightspace-tools', 'state-persistence'],
  },
  workflow: myAgentWorkflow,
});
```

If the agent loads data from Brightspace course files, declare them with `PluginInputDeclaration`:

```typescript
export const requiredInputs: PluginInputDeclaration[] = [
  {
    key: 'myData',
    type: 'json',
    sourceType: 'lms-file',      // reads from course files at runtime
    source: 'data/myData.json',  // path within the course files
    required: true,
  },
];
```

See [`frontend/src/agents/template-fidelity/config.ts`](frontend/src/agents/template-fidelity/config.ts) for a complete example.

---

## 5. The LLM Tool-Calling Loop Pattern

Most agents need a multi-turn LLM loop where the model calls Brightspace tools, receives results, and then produces structured output via a custom "submit" tool. This pattern is **not handled by the workflow engine** — you implement it directly in a service file.

The canonical implementation is [`frontend/src/agents/template-fidelity/services/fidelityAnalysis.ts`](frontend/src/agents/template-fidelity/services/fidelityAnalysis.ts).

```typescript
import {
  callLLMWithTools,
  getDefaultLLMClientOptions,
  getToolDefinitionsForLLM,
  loadApiKeyConfig,
  createUsageAccumulator,
  createBrowserAgentHostApi,
  interpolate,
  stripFrontmatter,
} from '@uga-brightspace/framework';
import type { ChatMessage, ToolCall } from '@uga-brightspace/framework';
import toolsConfig from '../data/tools.json';
import promptRaw from '../prompts/main.md?raw';

const MAX_TURNS = 20;

export async function runAnalysis(input: MyInput, context: MyContext) {
  // 1. Build tool list: Brightspace tools filtered by tools.json, plus any custom tools
  const disabledTools = (toolsConfig as { disabledTools?: string[] }).disabledTools ?? [];
  const toolDefs = [
    ...getToolDefinitionsForLLM(['content', 'grades', 'dropbox'], { disabledTools }),
    buildMySubmitTool(),   // custom output tool — see Section 6
  ];

  // 2. Build prompt (strip YAML frontmatter if present, interpolate variables)
  const systemPrompt = interpolate(stripFrontmatter(promptRaw), {
    courseName: input.courseName,
    ou: input.ou,
  });

  // 3. Initialize messages
  const messages: ChatMessage[] = [
    { role: 'system', content: systemPrompt },
    { role: 'user', content: 'Analyze the course and call submit_result when done.' },
  ];

  // 4. Create host API for executing Brightspace tools
  const hostApi = createBrowserAgentHostApi({
    ou: context.ou,
    xsrfToken: context.xsrfToken,
    coursePath: context.coursePath,
    courseDetails: context.courseDetails,
    whoAmI: null,
  });

  // 5. Track token usage
  const config = loadApiKeyConfig();
  const usage = createUsageAccumulator(config?.model ?? 'unknown');

  let capturedResult = null;

  // 6. Multi-turn loop
  for (let turn = 0; turn < MAX_TURNS; turn++) {
    const response = await callLLMWithTools(
      messages,
      toolDefs,
      undefined,
      getDefaultLLMClientOptions()
    );
    usage.add(response.usage);

    // No tool calls = LLM is done
    if (response.tool_calls.length === 0) break;

    messages.push({
      role: 'assistant',
      content: response.content,
      tool_calls: response.tool_calls,
    });

    for (const toolCall of response.tool_calls as ToolCall[]) {
      let result: string;

      if (toolCall.function.name === 'submit_result') {
        // Capture the structured output from the custom tool
        const args = JSON.parse(toolCall.function.arguments || '{}') as Record<string, unknown>;
        capturedResult = parseMyResult(args);
        result = JSON.stringify({ success: true });
      } else {
        // Execute a Brightspace tool
        try {
          const args = JSON.parse(toolCall.function.arguments || '{}') as Record<string, unknown>;
          result = await hostApi.executeTool(toolCall.function.name, args);
        } catch (err) {
          result = JSON.stringify({ error: err instanceof Error ? err.message : String(err) });
        }
      }

      messages.push({
        role: 'tool',
        content: result,
        tool_call_id: toolCall.id,
      });
    }

    // Exit loop once the LLM has submitted its result
    if (capturedResult) break;
  }

  return { result: capturedResult, usage: usage.snapshot() };
}
```

**Key points:**
- `getToolDefinitionsForLLM(categories, { disabledTools })` — converts Brightspace tool definitions to OpenAI function format. Always pass `disabledTools` from `data/tools.json`.
- `createBrowserAgentHostApi(context)` — creates an executor that routes tool calls to the Brightspace Valence API.
- `callLLMWithTools` — sends messages and tool definitions to the LLM. Returns `{ content, tool_calls, usage }`.
- The loop continues until the LLM produces no tool calls (finished thinking) or calls the custom submit tool.

---

## 6. Custom Output Tool Pattern

To receive structured output from the LLM (rather than free text), define a custom tool that the LLM calls when done. The framework calls these "submit tools".

```typescript
import type { ToolDefinition } from '@uga-brightspace/framework';

export const SUBMIT_RESULT_NAME = 'submit_result';

// Build the OpenAI-format tool definition
export function buildSubmitResultTool(): ToolDefinition {
  return {
    type: 'function',
    function: {
      name: SUBMIT_RESULT_NAME,
      description: 'Submit the analysis result. Call this once when done.',
      parameters: {
        type: 'object',
        properties: {
          summary: {
            type: 'string',
            description: 'Brief summary of findings.',
          },
          items: {
            type: 'array',
            items: {
              type: 'object',
              properties: {
                label: { type: 'string', description: 'Item label' },
                status: { type: 'string', enum: ['ok', 'issue'], description: 'Status' },
                detail: { type: 'string', description: 'Details. Use markdown.' },
              },
              required: ['label', 'status', 'detail'],
            },
          },
        },
        required: ['summary', 'items'],
      },
    },
  };
}

// Parse and validate the LLM's arguments
export function parseSubmitResult(args: Record<string, unknown>) {
  const summary = String(args.summary ?? '');
  const items = Array.isArray(args.items)
    ? args.items.map((i) => ({ label: String(i.label), status: i.status, detail: String(i.detail) }))
    : [];
  return { summary, items };
}
```

See [`frontend/src/agents/template-fidelity/tools/submitFidelityReport.ts`](frontend/src/agents/template-fidelity/tools/submitFidelityReport.ts) for a complete example.

---

## 7. State Persistence

To save and load data from Brightspace course files, use `brightspaceClient.readCourseFile`, `createCourseFolder`, and `uploadFileToCourse`.

```typescript
import { brightspaceClient, createCourseFolder, uploadFileToCourse } from '@uga-brightspace/framework';
import type { MyReport } from '../types.js';

const FOLDER = 'my-agent';
const FILE = 'results.json';

export async function loadResults(ou: string): Promise<MyReport[]> {
  const data = await brightspaceClient.readCourseFile<{ results?: MyReport[] }>(ou, `${FOLDER}/${FILE}`);
  return data?.results ?? [];
}

export async function saveResults(ou: string, xsrfToken: string, results: MyReport[]): Promise<void> {
  await createCourseFolder(ou, FOLDER, xsrfToken);
  await uploadFileToCourse(ou, FILE, JSON.stringify({ results }, null, 2), FOLDER, xsrfToken, true);
}
```

See [`frontend/src/agents/template-fidelity/services/reportsStorage.ts`](frontend/src/agents/template-fidelity/services/reportsStorage.ts) for a complete example.

---

## 8. Reading Data from Brightspace Course Files (lms-file)

When an agent needs to read a JSON file that lives in the course's file manager (not the LLM tool system), use `brightspaceClient.readCourseFile` directly in a service function:

```typescript
import { brightspaceClient } from '@uga-brightspace/framework';
import type { MyData } from '../types.js';

export async function loadMyData(ou: string): Promise<MyData> {
  const data = await brightspaceClient.readCourseFile<MyData>(ou, 'data/myData.json');
  if (!data) throw new Error('data/myData.json not found in course files');
  return data;
}
```

Declare the input in `config.ts` so the framework knows about it:

```typescript
export const requiredInputs: PluginInputDeclaration[] = [
  {
    key: 'myData',
    type: 'json',
    sourceType: 'lms-file',
    source: 'data/myData.json',
    required: true,
  },
];
```

See [`frontend/src/agents/template-fidelity/services/homepageDataService.ts`](frontend/src/agents/template-fidelity/services/homepageDataService.ts) for a complete example.

---

## 9. Framework Exports Reference

All exports from `@uga-brightspace/framework` (source: [`frontend/src/agent-api/index.ts`](frontend/src/agent-api/index.ts)):

### Workflow / Plugin
| Export | Purpose |
|--------|---------|
| `workflow(id)` | Builder API for workflow definition |
| `defineAgentPlugin({ manifest, workflow })` | Creates the agentPlugin manifest object |
| `loadAgentPlugin({ agentPlugin })` | Called in entry.tsx to register the plugin |

### Brightspace
| Export | Purpose |
|--------|---------|
| `brightspaceClient` | Typed wrapper over Valence API (`readCourseFile`, `fetchOrgUnitChildren`, etc.) |
| `useBrightspace()` | React hook — returns `{ ou, xsrfToken, coursePath, courseDetails, whoAmI, courseRole, isLoading }` |
| `BrightspaceProvider` | React context provider — wraps the app in entry.tsx |
| `createCourseFolder(ou, folder, xsrfToken)` | Creates a folder in course files |
| `uploadFileToCourse(ou, filename, content, folder, xsrfToken, overwrite)` | Writes a file to course files |
| `createFolderAndUploadFile(...)` | Combined create + upload |
| `parseSemesterFromOrgUnitCode(code)` | Parses "Fall 2024" from a Brightspace org unit code like `CO.180.X.12345.20252` |

### LLM
| Export | Purpose |
|--------|---------|
| `callLLMWithTools(messages, tools, options, clientOptions)` | Send messages to LLM, returns `{ content, tool_calls, usage }` |
| `callLLMStream(messages, options, clientOptions, onChunk)` | Streaming variant |
| `getDefaultLLMClientOptions()` | Returns `{ mode: 'proxy', proxyUrl }` or `{ mode: 'direct' }` from env |
| `loadApiKeyConfig()` | Returns `{ provider, model, apiKey }` from localStorage |
| `createUsageAccumulator(model)` | Accumulates token usage across turns, call `.add(usage)` and `.snapshot()` |
| `formatCost(dollars)` | Format as "$0.12" or "<$0.01" |
| `formatTokens(n)` | Format as "12.3k" or "1.2M" |
| `mergeUsage(a, b)` | Merge two UsageSnapshot objects |

### Tools
| Export | Purpose |
|--------|---------|
| `getToolDefinitionsForLLM(categories?, options?)` | Returns OpenAI-format tool list. Pass `{ disabledTools }` from tools.json |
| `getToolByName(name)` | Looks up a Brightspace tool definition by name |
| `executeBrightspaceTool(name, args, context)` | Execute a single tool directly (without the LLM loop) |
| `registerAllTools()` | Call once in entry.tsx to load all builtin tool definitions into the registry |
| `createBrowserAgentHostApi(context)` | Creates executor for the LLM loop. Context: `{ ou, xsrfToken, coursePath, courseDetails, whoAmI }` |

### Prompts
| Export | Purpose |
|--------|---------|
| `interpolate(template, vars)` | Replace `{{key}}` and `{{nested.key}}` in a string |
| `stripFrontmatter(md)` | Remove YAML frontmatter from a markdown string |
| `parseFrontmatter(md)` | Extract YAML frontmatter as an object |

### Data Loading
| Export | Purpose |
|--------|---------|
| `loadInput(declaration, context)` | Load a single declared input at runtime |
| `loadInputs(declarations, context)` | Load all declared inputs |
| `loadCatalog(declaration, context, options?)` | Load catalog items for dropdowns (lms-catalog) |
| `validateInputs(bag, declarations)` | Validate loaded inputs against Zod schemas |

### Key Types
```typescript
import type {
  ChatMessage,        // { role, content, tool_calls?, tool_call_id? }
  ToolCall,           // { id, function: { name, arguments } }
  ToolDefinition,     // OpenAI function definition format
  PluginInputDeclaration,
  OrgUnitChild,       // { Identifier, Name, Code, Path, Type }
  UsageSnapshot,      // { promptTokens, completionTokens, totalTokens, estimatedCost, model, calls }
  ToolCategory,       // 'content' | 'course' | 'discussions' | 'grades' | 'quizzes' | 'dropbox' | 'rubrics' | 'other'
} from '@uga-brightspace/framework';
```

---

## 10. UI Components

### Required shell structure (already in scaffolded entry.tsx)

Every agent entry.tsx wraps the app in `BrightspaceProvider` and renders `AgentShell`:

```tsx
import { AgentShell, ProviderSettings } from '../../ui/index.js';

<AgentShell title="My Agent" headerRight={<button ...>Settings</button>}>
  <MainView />
  <ProviderSettings isOpen={settingsOpen} onClose={() => setSettingsOpen(false)} />
</AgentShell>
```

`AgentShell` provides the header bar and main content area. `ProviderSettings` is the modal for configuring the LLM provider and model — always include it.

### Available UI components (from `../../ui/index.js`)

| Component | Purpose |
|-----------|---------|
| `AgentShell` | Root layout shell with header bar |
| `ProviderSettings` | LLM provider settings modal |
| `Modal` | Generic modal overlay |
| `ChatPanel` | Chat message list with input |
| `WorkflowProgress` | Multi-step progress indicator |
| `StepIndicator` | Single step circle |
| `ApprovalGate` | User approval checkpoint UI |
| `StagingPreview` | Preview before committing output |
| `InputForm` | Labeled form fields |
| `FileUpload` | File upload widget |
| `OutputDisplay` | Formatted agent output display |
| `StatusBadge` | Colored status label |
| `UsageDisplay` | Token count and cost display |
| `ProgressBar` | Progress bar |
| `ErrorBoundary` | React error boundary wrapper |

### Scrollable main content

The `agent-shell__main` has `overflow: hidden`. For scrollable content, wrap your view in `agent-scroll-pane`:

```tsx
<div className="agent-scroll-pane">
  {/* scrollable content */}
</div>
```

For a flex column that scrolls its list while keeping a fixed header, use:

```tsx
<div className="util-pad-all-md agent-flex-col-full" style={{ minHeight: 0 }}>
  <div style={{ flexShrink: 0 }}>/* fixed header */</div>
  <div style={{ flex: 1, minHeight: 0, overflowY: 'auto' }}>/* scrolling content */</div>
</div>
```

---

## 11. Design System Rules

All agent UI must use the **UGA Online Design System**.

### Class prefixes

| Prefix | Purpose | Examples |
|--------|---------|---------|
| `cmp-*` | Components | `cmp-button`, `cmp-button--red`, `cmp-button--white`, `cmp-button--narrow`, `cmp-heading-2`, `cmp-heading-5`, `cmp-heading-6`, `cmp-paragraph`, `cmp-badge`, `cmp-badge--green`, `cmp-badge--yellow`, `cmp-badge--red`, `cmp-form-field__input`, `cmp-label`, `cmp-list` |
| `obj-*` | Layout objects | `obj-flex`, `obj-flex--align-items__center`, `obj-flex--justify-content__between`, `obj-reading-width` |
| `util-*` | Utilities | `util-pad-all-md`, `util-pad-all-sm`, `util-background-white`, `util-color-red` |

### Framework-specific `agent-*` classes (from `frontend/src/index.css`)

| Class | Purpose |
|-------|---------|
| `agent-shell` | Root shell layout (used by AgentShell) |
| `agent-shell__header` | Header bar |
| `agent-shell__main` | Main content area (overflow: hidden) |
| `agent-scroll-pane` | Scrollable full-height pane with padding |
| `agent-flex-col-full` | `display:flex; flex-direction:column; height:100%` |
| `agent-flex-1` | `flex: 1; min-width: 0` |
| `agent-flex-gap-sm` | Flex with 0.5rem gap |
| `agent-flex-gap-md` | Flex with 1rem gap |
| `agent-gap-xs/sm/md/lg` | Gap utilities (0.25/0.5/0.75/1rem) |
| `agent-card` | Bordered card with padding and white background |
| `agent-card__row` | Flex row inside a card with bottom border |
| `agent-text-sm/xs/xxs/base/md` | Font size helpers |
| `agent-text-muted/subtle/faint` | Muted text colors |
| `agent-section-label` | Uppercase section heading label |
| `agent-spinner` | Animated loading spinner |
| `agent-center` | Centered full-height loading layout |
| `agent-text-center` | Centered text with padding |
| `agent-error-box` | Red-bordered error message box |
| `agent-running-box` | Background box for running state |
| `agent-modal-overlay` | Fixed modal backdrop |
| `agent-progress-bar__track` | Progress bar track |
| `agent-progress-bar__fill` | Progress bar fill (set width as percentage) |
| `agent-button-reset` | Unstyled transparent button |
| `agent-disabled` | Dimmed, non-interactive state |
| `agent-inline-label` | Inline checkbox/icon label |
| `agent-badge--sm` | Small badge modifier |
| `agent-badge--purple` | Purple badge color |

### CSS variables

Always use CSS variables for colors — never hardcode hex values for brand colors:

```
--uga-red       #ba0c2f
--uga-lake      #00a3ad
--uga-hedges    #b7bf10
--uga-olympic   #004e60
--uga-gray      #dedede
--uga-light-gray  #f2f2f2
--uga-dark-gray   #666666
```

### Rules
- **Do not** use Tailwind or arbitrary utility classes — the framework does not include Tailwind
- **Do not** hardcode hex colors that should be CSS variables
- **Do not** add `<style>` blocks in components — write CSS in `index.css` using `agent-*` prefix if a new pattern is needed

---

## 12. Tool Categories and tools.json

`data/tools.json` controls which Brightspace tools are sent to the LLM. Keep it minimal — more tools = larger request payloads.

```json
{
  "disabledTools": [
    "get_module_access",
    "get_content_pacing",
    "get_dropbox_submissions",
    "get_classlist"
  ]
}
```

Available tool categories:

| Category | What it provides |
|----------|-----------------|
| `content` | TOC, modules, topics, file reading |
| `course` | Course details, sections, groups, course files |
| `discussions` | Forums, topics, posts |
| `grades` | Grade objects, categories, schemes, setup |
| `quizzes` | Quizzes, questions |
| `dropbox` | Assignment folders |
| `rubrics` | Rubrics, competencies |
| `other` | News, calendar, checklists, surveys, LTI links |

The full list of tool names within each category is in `frontend/src/core/tools/builtins/`. The Tool Editor UI (dev mode) lets you toggle tools on/off and saves to `data/tools.json` automatically.

---

## 13. Conformance Checklist

Before considering an agent complete, verify:

- [ ] All imports use `@uga-brightspace/framework` — no deep `core/*` imports
- [ ] `agentPlugin` exported from `config.ts` with `pluginApiVersion` and `frameworkApiVersion`
- [ ] `agentPlugin` re-exported from `index.ts`
- [ ] `entry.tsx` loads `registerAllTools()` and `loadAgentPlugin({ agentPlugin })`
- [ ] UI uses UGA design system classes (`cmp-*`, `obj-*`, `util-*`) — no Tailwind
- [ ] `data/tools.json` has a `disabledTools` array (may be empty)
- [ ] Prompt templates are `.md` files in `prompts/` with `{{variable}}` placeholders
- [ ] TypeScript types are defined in `types.ts`
- [ ] State saved to Brightspace uses `createCourseFolder` + `uploadFileToCourse`
- [ ] `AgentShell` is the root layout component in `entry.tsx`
- [ ] `ProviderSettings` is included in `entry.tsx` for LLM configuration
- [ ] Build passes: `npm run build:agent -- my-agent`
