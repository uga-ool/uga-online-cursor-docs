# Brightspace Agent Framework — Architecture

> **For AI coding agents implementing a new agent plugin:** read [AGENT-GUIDE.md](AGENT-GUIDE.md) first. It is a focused, task-oriented reference that consolidates everything you need. Come back here for deeper architecture context when needed.

This document describes the mental model, core systems, and extension points for the framework.

## Scope

**This repo contains only the framework.** It does not include agent implementations (e.g., Course Review, Module Builder). Those live in separate app repos. This repo provides:

- Architecture and protocols
- Core systems (workflow, tools, state, prompts, data loading)
- UI component library
- Agent scaffolding script (`create-agent`)
- Instructions for creating and implementing agents

## Overview

The framework provides a reusable foundation for AI-powered agent applications that run inside Brightspace LMS. Agents are **workflow-driven**, not conversation-driven. They consume structured inputs (prompts, JSON schemas, uploaded files, LMS data) and produce structured outputs (course content, review findings, reports).

## Repository Structure

```
uga-online-brightspace-agent-framework/
├── backend/          # Thin API proxy (API key management, LLM forwarding)
├── frontend/         # React app + framework core
│   └── src/
│       ├── core/     # Framework core (workflow, tools, brightspace, llm, state, prompts)
│       ├── ui/       # Reusable UI components
│       └── agents/   # Empty — add agents via create-agent (in consuming apps)
├── shared/           # TypeScript types and Zod schemas
└── scripts/         # create-agent scaffolding
```

## Core Systems

### 1. Workflow Engine

Orchestrates multi-step AI-powered processes.

- **Builder API**: `workflow('id').name().input().ai().gate().forEach().build()`
- **Step kinds**: `ai` (LLM + tools), `tool` (direct tool call), `transform` (data), `gate` (user approval), `forEach` (iteration), `branch` (conditional)
- **State**: Persisted to Brightspace course files via the state layer
- **Execution**: `executeWorkflow(definition, input, context)` runs steps in order

**Key files**: `core/workflow/builder.ts`, `core/workflow/engine.ts`, `core/workflow/context.ts`

### 2. Tool System

Standardized function-calling for AI models.

- **Registry**: `registerTools()`, `getToolByName()`, `getToolsForCategories()`
- **Brightspace tools**: Declarative `BrightspaceToolDef` with `pathTemplate`, `params`, `responseType`
- **Execution**: `executeBrightspaceTool(name, args, context)` resolves URL, calls API, filters response
- **Response filter**: Schema-based whitelist in `fieldSchemas.json` reduces token usage
- **Adapter**: `getToolDefinitionsForLLM(categories)` converts to OpenAI function format

**Key files**: `core/tools/registry.ts`, `core/tools/executor.ts`, `core/tools/responseFilter.ts`, `core/tools/builtins/`

### 3. Brightspace Integration

- **Client**: `brightspaceClient` — typed wrapper over Valence API (courses, files, content, enrollments)
- **Context**: `BrightspaceProvider` / `useBrightspace()` — ou, xsrfToken, coursePath, courseDetails, whoAmI, courseRole
- **Files**: `createCourseFolder()`, `uploadFileToCourse()` — course file operations
- **Content**: Module/topic creation (see reference app patterns)

**Key files**: `core/brightspace/client.ts`, `core/brightspace/context.tsx`, `core/brightspace/files.ts`

### 4. LLM Client

- **Direct mode**: `fetch()` to OpenAI/OpenRouter/Airia with client-side API keys
- **Proxy mode**: Forwards to backend; keys stored server-side
- **Providers**: `loadApiKeyConfig()`, `saveApiKeyConfig()` — multi-provider support
- **Usage**: `createUsageAccumulator()`, `formatCost()`, `formatTokens()`

**Key files**: `core/llm/client.ts`, `core/llm/providers.ts`, `core/llm/tokenTracker.ts`

### 5. State Persistence

- **Envelope**: `StateEnvelope<T>` — version, agentId, workflowId, courseOu, status, context, data
- **Persistence**: `loadState()`, `saveState()`, `updateState()` — Brightspace course files
- **Serialization**: Per-key queue (`enqueueSave`) avoids concurrent write conflicts

**Key files**: `core/state/envelope.ts`, `core/state/persistence.ts`

### 6. Prompt Management

- **Templates**: Markdown files with `{{variable}}` placeholders
- **Interpolation**: `interpolate(template, variables)` — dot-path resolution
- **Composition**: `composePrompt(layer, variables)` — system + agent + step + context

**Key files**: `core/prompts/manager.ts`, `core/prompts/composer.ts`

### 7. Context Ingestion

- **Declaration**: `InputDeclaration` — key, type, source, schema, required
- **Loading**: `loadInput(declaration, loaders)` — JSON, markdown, LMS data
- **Validation**: `validateInputs(bag, declarations)` — Zod schema validation

**Key files**: `core/context/ingestion.ts`

## Agent Structure

```
agents/my-agent/
├── config.ts          # workflow definition, agent metadata
├── entry.tsx          # app entrypoint for building this agent
├── entry.html         # deployable HTML loader for this agent bundle
├── workflow.ts        # (optional) workflow in separate file
├── prompts/           # Markdown prompt templates
├── components/        # Custom UI components
├── tools/             # Custom tools (beyond builtins)
├── data/              # JSON config, schemas
└── index.ts           # Public exports
```

## Plugin Contract (Portability)

Agents are now treated as plugins with a manifest contract shared via `@uga-brightspace/shared`:

- `agentPlugin.manifest.id`, `name`, `version`
- `pluginApiVersion` and `frameworkApiVersion` compatibility handshake
- Optional `capabilities` and `requiredInputs`

Host implementations validate `agentPlugin` at load time and reject incompatible versions.

### Stable Agent API

Agent code should import framework functionality from:

- `@uga-brightspace/framework`

Avoid deep imports into `frontend/src/core/*` from agent code. The framework export surface is the portability boundary.

## Creating a New Agent

1. Run `npm run create-agent -- --name "my-agent"`
2. Edit `config.ts` to define workflow steps, input schema, and `agentPlugin` manifest
3. Add prompt templates in `prompts/`
4. Build custom components in `components/`
5. Register in `App.tsx` or agent router

## Conventions

- **Naming**: Agent IDs use kebab-case; workflow step IDs use camelCase
- **State files**: `{folder}/{prefix}_{id}.json` in course files
- **Tool categories**: `content`, `course`, `discussions`, `grades`, `quizzes`, `dropbox`, `rubrics`, `other`
- **Provider URLs**: OpenAI, OpenRouter, Airia (all OpenAI-compatible)

## Design System

All agent UIs must use the **UGA Office of Online Learning Design System**: https://design.online.uga.edu

- **CSS**: Load `https://design.online.uga.edu/css/base.css` (included in agent entry HTML)
- **JS**: Load `https://design.online.uga.edu/js/scripts.js` for tabs, accordions, etc.
- **Fonts**: Merriweather Sans (body), Merriweather (serif headings), Oswald (uppercase headings)

### Class Naming

| Prefix | Purpose | Examples |
|--------|---------|----------|
| `cmp-*` | Components | `cmp-button`, `cmp-heading-2`, `cmp-form-field__input`, `cmp-paragraph` |
| `obj-*` | Layout objects | `obj-flex`, `obj-flex--justify-content__between`, `obj-reading-width` |
| `util-*` | Utilities | `util-pad-all-md`, `util-background-white`, `util-color-red` |

### Framework-Specific Classes

The framework adds `agent-*` classes in `app.css` for patterns the UGA design system does not provide:

- `agent-modal-overlay` — modal backdrop
- `agent-progress-bar` — progress bar track/fill
- `agent-spinner` — loading spinner
- `agent-center` — centered content
- `agent-text-center` — centered text
- `agent-step-circle` — workflow step indicator
- `agent-step-connector` — step connector line

### Usage Rules

- **Do** use UGA design system classes for typography, buttons, forms, layout, spacing
- **Do** use `agent-*` classes only for framework-specific UI (modal, spinner, progress, workflow steps)
- **Do not** use Tailwind or arbitrary utility classes; the framework does not include Tailwind
- **Reference**: See design system docs for Components, Elements, Utilities

## Extending the Framework

- **Custom tools**: Implement `ToolDefinition` with Zod schema and `execute()`; register with `registerTools()`
- **Brightspace tools**: Add to `core/tools/builtins/`; add schema to `fieldSchemas.json`
- **UI components**: Add to `ui/`; use UGA design system classes
- **Workflow steps**: Extend `StepKind` and add step execution in `engine.ts`

## Backend Proxy

- **Endpoint**: `POST /api/llm/chat`, body: `{ messages, tools?, model?, temperature?, maxTokens? }`
- **Config**: `LLM_PROVIDER`, `LLM_API_KEY`, `LLM_MODEL` from env
- **Session**: Optional `REQUIRE_BRIGHTSPACE_SESSION=true` to enforce X-Csrf-Token
- **Rate limit**: 60 requests/minute per IP (configurable)

## Reference

- **Course Review app**: Separate repo — full example with workflow, StandardsView, QM/OSCQR data
