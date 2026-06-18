# Agents

This folder is for agent implementations. Each agent is built and deployed as a separate Brightspace tool.

## Adding an agent

From the project root:

```bash
npm run create-agent -- --name "your-agent"
```

This scaffolds `agents/your-agent/` with:

- `config.ts` — workflow definition, agent metadata, and `agentPlugin` manifest
- `prompts/main.md` — prompt template
- `components/MainView.tsx` — main view component
- `entry.tsx` — app entrypoint used for agent build input
- `entry.html` — deployable loader HTML for the built agent bundle
- `index.ts` — public exports

## Developing an agent

Run the dev server with authoring tools (Schema Editor, Prompt Editor):

```bash
npm run dev:agent -- your-agent
```

Opens http://localhost:5173/dev.html with tabs: Agent | Schema Editor | Prompt Editor. Edit JSON in `data/` and markdown in `prompts/` directly. Agent functionality may not work locally (requires Brightspace).

## Building an agent

From the frontend directory (or project root with `--workspace=frontend`):

```bash
# Production build (output: dist_zip/your-agent-prod.zip)
npm run build:agent -- your-agent

# Production build for Shared deployment (app-shared-base baked in)
npm run build:agent -- your-agent --shared

# Development build (output: dist_zip/your-agent-dev.zip)
npm run build:agent:dev -- your-agent

# Build all agents
npm run build:all
npm run build:all:dev
```

The zip is ready for upload to Brightspace.

## Deployment

**Colocated**: Upload the full zip (`<agent>.html` + assets/) to a course folder. The loader finds assets in the same directory. No configuration needed.

**Shared**: When the entry HTML is in a course but assets live in Shared (public) storage, set `app-shared-base` at build time so the loader knows where to find assets. Priority: shell env > `--shared` flag > agent `.env`.

```bash
# Option 1: Agent .env (persistent) — add to src/agents/your-agent/.env:
APP_SHARED_BASE=/shared/ugaonline/apps/your-agent/

# Option 2: --shared flag (convention: /shared/ugaonline/apps/<agent>/)
npm run build:agent -- your-agent --shared

# Option 3: Shell env (one-off override)
APP_SHARED_BASE=/shared/ugaonline/apps/your-agent/ npm run build:agent -- your-agent
```

The value must end with a trailing slash and point to the Shared folder containing the assets/ directory. Upload `<agent>.html` to the course; upload assets/ to the Shared path. Configure the LTI launch URL to point at the HTML file.

## Design system

All agent UIs must use the **UGA Online Design System**: https://design.online.uga.edu

- Use `cmp-*` (components), `obj-*` (layout), `util-*` (utilities) classes
- Use `agent-*` classes only for framework-specific UI (spinner, modal, progress bar)
- Do not use Tailwind or arbitrary utility classes

See ARCHITECTURE.md § Design System for full reference.

## Data loading

Declare inputs in `config.ts` with `PluginInputDeclaration` and `sourceType`:

| sourceType | When | Use case |
|------------|------|----------|
| `bundle` | Build-time | Static JSON/MD in agent package |
| `lms-file` | Runtime | Single file from Brightspace course files |
| `lms-catalog` | Runtime | List folder for dropdown (e.g. user-uploaded rubrics) |
| `url` | Runtime | External URL |
| `lms-data` | Runtime | LMS context (courseDetails, whoAmI) |

Use `loadInput()` and `loadCatalog()` from framework APIs. For `lms-catalog` with `bundleFallback`, built-in items merge with user-uploaded for dropdowns.

## Portability Contract

Agents must import framework APIs from `@uga-brightspace/framework` (stable public surface), not deep `core/*` paths.

- Export `agentPlugin` from `config.ts` using `defineAgentPlugin(...)`.
- Include a manifest with `pluginApiVersion` and `frameworkApiVersion`.
- Export `agentPlugin` from `index.ts` so host loaders can validate compatibility.
- Keep `entry.tsx` and `entry.html` inside the agent folder so the entire agent can be shared as one directory.

## Conformance checklist

Before considering an agent complete, verify:

- [ ] Uses UGA design system classes (`cmp-*`, `obj-*`, `util-*`) — no Tailwind
- [ ] Declares `requiredInputs` with correct `sourceType` for all data inputs
- [ ] Uses `AgentShell` and framework UI components
- [ ] Workflow defined via builder API in config.ts
- [ ] State folder and file naming follow conventions
