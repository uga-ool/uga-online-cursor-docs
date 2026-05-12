# Brightspace Agent Framework — Developer Guide

> **This guide is for human developers and designers.** If you are an AI coding agent, read [AGENT-GUIDE.md](AGENT-GUIDE.md) instead.

This guide covers environment setup, scaffolding a new agent, local development, building for deployment, and deploying to Brightspace.

---

## 1. Prerequisites

- **Node.js 20+** and **npm** installed locally
- Access to a Brightspace LMS instance (required for all API calls — local dev mode is UI-only)
- An LLM provider key **or** access to the UGA proxy backend (for AI-powered features)

---

## 2. Environment Setup

Copy or create a `.env` file in the `frontend/` directory. The following variables are supported:

```bash
# Brightspace Valence API (required for API calls when deployed inside Brightspace)
VITE_API_BASE_URL=/d2l/api
VITE_LP_VERSION=1.54
VITE_LE_VERSION=1.76

# LLM mode: "proxy" (routed via backend) or "direct" (client talks to provider directly)
VITE_LLM_MODE=proxy

# Required when VITE_LLM_MODE=proxy — URL of the backend API gateway
VITE_BACKEND_URL=https://<your-gateway-id>.execute-api.<region>.amazonaws.com

# Optional streaming endpoint
VITE_BACKEND_STREAM_URL=https://<your-streaming-url>.lambda-url.<region>.on.aws/
```

For local development, the API variables don't matter — LMS-dependent features only work when the tool is embedded in Brightspace as an LTI widget.

---

## 3. Create a New Agent

From the `frontend/` directory (or the repo root using `--workspace=frontend`):

```bash
npm run create-agent -- --name "my-agent"
```

This scaffolds `frontend/src/agents/my-agent/` with the following files:

| File | Purpose |
|------|---------|
| `config.ts` | Workflow definition, agent metadata, and plugin manifest |
| `prompts/main.md` | Primary LLM system prompt template |
| `components/MainView.tsx` | Main UI component |
| `entry.tsx` | App entry point (used as the Vite build input) |
| `entry.html` | Deployable HTML loader for the built bundle |
| `index.ts` | Public exports |

After scaffolding, open `config.ts` and update the agent name, description, and tool categories. Then write your logic in `components/MainView.tsx` and add services as needed.

---

## 4. Run Locally (Dev Mode)

```bash
npm run dev:agent -- my-agent
```

Opens [http://localhost:5173/dev.html](http://localhost:5173/dev.html) with three tabs:

| Tab | What it does |
|-----|-------------|
| **Agent** | Renders the agent UI. LMS-dependent features (Brightspace API calls, state persistence) will not work here. |
| **Schema Editor** | Edit `data/schema.json` — the input schema the agent validates against. |
| **Prompt Editor** | Edit `prompts/*.md` files — the LLM system prompt templates. |

You can still test layout, form interactions, and non-LMS logic locally.

---

## 5. Configure Tools (Tool Editor)

In dev mode, there is a **Tool Editor** tab available in the developer toolbar. Use it to:

1. Browse all available Brightspace tool categories and individual tools.
2. Toggle tools on or off.
3. Click **Save** — this writes the tool selection to `data/tools.json`.

Fewer enabled tools = smaller LLM request payloads. Only enable tools the agent actually needs.

The tool configuration is read at runtime from `data/tools.json` and passed to the LLM. If the agent is hitting payload size errors (HTTP 413), disable unused tools via the Tool Editor.

---

## 6. Build for Deployment

All build commands are run from the `frontend/` directory.

### Production build (colocated)

```bash
npm run build:agent -- my-agent
```

Output: `frontend/dist_zip/my-agent-prod.zip`

### Production build (shared storage)

Use this when agent HTML lives in a Brightspace course but the JavaScript assets are hosted in Brightspace Shared storage (to avoid re-uploading assets to every course).

```bash
npm run build:agent -- my-agent --shared
```

Or set the shared path in an agent-specific `.env` file at `src/agents/my-agent/.env`:

```bash
APP_SHARED_BASE=/shared/ugaonline/apps/my-agent/
```

### Development build

```bash
npm run build:agent:dev -- my-agent
```

Output: `frontend/dist_zip/my-agent-dev.zip` (unminified, includes source maps)

### Build all agents

```bash
npm run build:all          # all agents, production
npm run build:all:dev      # all agents, dev build
```

---

## 7. Deploy to Brightspace

### Option A — Colocated (recommended for most agents)

1. Unzip `my-agent-prod.zip`. You'll find `my-agent.html` and an `assets/` folder.
2. In Brightspace, open the target course → **Course Admin** → **Manage Files**.
3. Create a folder (e.g. `my-agent/`) and upload both `my-agent.html` and the `assets/` folder into it.
4. In **External Learning Tools**, register a new tool:
   - **Launch URL**: the full URL to `my-agent.html` in the course files, e.g.  
     `https://your-institution.brightspace.com/content/enforced/<OU>/my-agent/my-agent.html`
   - **Launch type**: New window or iframe, depending on your setup.
5. Add the LTI tool to a course page or the navigation bar.

### Option B — Shared storage

1. Upload the `assets/` folder to the Brightspace Shared path you configured (e.g. `/shared/ugaonline/apps/my-agent/assets/`).
2. Upload only `my-agent.html` to each course that needs the tool.
3. Register the LTI tool pointing to the per-course HTML path.

**The shared base path must end with a trailing slash** and must match the path set in `APP_SHARED_BASE` at build time.

---

## 8. Requesting an Agent from a Coding Agent

If you want an AI coding assistant to implement an agent for you, use the template in [AGENT-REQUEST-TEMPLATE.md](AGENT-REQUEST-TEMPLATE.md). Fill in the sections describing what the agent should do, what Brightspace data it needs, and what the LLM should produce — then paste the filled template into the coding agent conversation.

The coding agent will read [AGENT-GUIDE.md](AGENT-GUIDE.md) to understand the framework conventions before it starts writing code.

---

## 9. Troubleshooting: "Not connected to service or token expired" (Asana, Google Drive, etc.)

When the agent reports it doesn't have access to an external service (e.g. Asana) even though you connected it in Settings, the backend is returning 401 because it cannot find valid OAuth tokens. Work through this checklist:

### 1. Same backend for connect and run

- **Connect** (OAuth) and **proxy** (tool calls) both use `VITE_BACKEND_URL`.
- Ensure the frontend `.env` points to the same backend where you completed OAuth.
- If you connected while pointing at a different backend (e.g. local SAM vs deployed API), tokens are stored in that backend's DynamoDB — the agent's backend has no tokens.

### 2. Same user context

- Tokens are stored by `userId` (Brightspace `whoAmI.Identifier`).
- Connect and run must use the **same** Brightspace user. If you connected in one course/context and run in another, the Identifier should still match for the same user.
- If the agent runs **without Brightspace context** (e.g. dev mode at `localhost:5173` without an LTI launch), `userId` is undefined and you'll see "User context required for service tools" instead.

### 3. Backend configuration

- **DynamoDB**: The backend needs `DYNAMODB_TABLE_NAME` set to the service-tokens table (SAM creates `{StackName}-service-tokens`).
- **OAuth credentials**: For Asana, set `AsanaClientId` and `AsanaClientSecret` in the SAM template or backend env.
- **CORS**: `AllowedOrigins` in the SAM template must include the origin where the agent runs (e.g. `https://ugatest2.view.usg.edu`).

### 4. Reconnect after environment changes

If you changed backend URL, deployed a new stack, or switched environments, **disconnect and reconnect** the service in Settings. Tokens are stored per backend.

### 5. Verify connection status

In the agent's Settings → Services tab, the status badge should show "Connected" for Asana. If it shows "Not connected" after you thought you connected, the backend `checkStatus` call is not finding tokens — usually due to userId or backend mismatch.

---

## 10. Useful npm Scripts Summary

| Command | What it does |
|---------|-------------|
| `npm run create-agent -- --name <name>` | Scaffold a new agent directory |
| `npm run dev:agent -- <name>` | Start local dev server for one agent |
| `npm run dev` | Start the framework-level dev server (not agent-specific) |
| `npm run build:agent -- <name>` | Production build for one agent |
| `npm run build:agent:dev -- <name>` | Dev build for one agent |
| `npm run build:all` | Production build for all agents |
| `npm run build:all:dev` | Dev build for all agents |
| `npm run lint` | Run ESLint and agent import guard |
| `npm run test` | Run all unit tests |
