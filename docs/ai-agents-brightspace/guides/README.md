# UGA Brightspace Agent Framework

Framework for building AI-powered Brightspace tools (agents) with workflow orchestration, LLM integration, and the UGA design system.

## Quickstart

### 1. Create an agent

From the project root:

```bash
npm run create-agent -- --name "my-agent"
```

This scaffolds `frontend/src/agents/my-agent/` with config, prompts, components, data, and colocated entry files (`entry.tsx`, `entry.html`) used for build and deployment.

Scaffolded agents use the stable framework import surface `@uga-brightspace/framework` and export an `agentPlugin` manifest for compatibility checks.
Each scaffolded agent is self-contained under `frontend/src/agents/<agent>/` including `entry.tsx` and `entry.html` for easier sharing.

### 2. Run locally

From the project root:

```bash
npm run dev:agent --workspace=frontend -- my-agent
```

Or from `frontend/`:

```bash
npm run dev:agent -- my-agent
```

Opens http://localhost:5173/dev.html with:

- **Agent** tab — the agent UI (may show "no Brightspace" when run locally)
- **Schema Editor** — edit JSON files in `agents/my-agent/data/`
- **Prompt Editor** — edit markdown files in `agents/my-agent/prompts/`

Agent functionality may not work locally without Brightspace and LLM API access, but you can develop and author schemas and prompts.

### 3. Build

From the project root:

```bash
# Production build
npm run build:agent --workspace=frontend -- my-agent

# Development build (for testing)
npm run build:agent:dev --workspace=frontend -- my-agent

# Build all agents
npm run build:all --workspace=frontend
```

Or from `frontend/`:

```bash
npm run build:agent -- my-agent
npm run build:agent:dev -- my-agent
npm run build:all
```

Output: `frontend/dist_zip/my-agent-prod.zip` (or `-dev.zip`).

### 4. Deploy to Brightspace

1. Upload `frontend/dist_zip/my-agent-prod.zip` to Brightspace:
   - **Colocated**: Upload the full zip (`<agent>.html` + assets/) to a course folder or LOR. No configuration needed.
   - **Shared**: When the entry HTML lives in a course but assets live in Shared storage, build with `--shared` or `APP_SHARED_BASE=/shared/your-org/apps/my-agent/` so `app-shared-base` is baked in. Upload `<agent>.html` to the course; upload the assets/ folder to the Shared path. Configure the LTI launch URL to point at the HTML file.
2. Configure the tool in Brightspace Admin → External Learning Tools (or equivalent) to register the agent as an LTI tool.
3. Ensure the frontend `.env` (or build-time env) has:
   - **Brightspace API**: `VITE_API_BASE_URL`, `VITE_LP_VERSION`, `VITE_LE_VERSION` — required for Valence API calls (see `frontend/.env.example`)
   - **LLM**: `VITE_LLM_MODE=proxy` and `VITE_BACKEND_URL` — for the agent to use your proxy for LLM calls
   - **Streaming** (optional): `VITE_BACKEND_STREAM_URL` — Lambda Function URL for streaming chat responses. If unset, the frontend uses `VITE_BACKEND_URL/api/llm/chat/stream` (works for local dev; for production, set to the Stream URL output after deploy).

For backend deployment:

```bash
cd backend
sam build && sam deploy
```

See `backend/README.md` (if present) or `backend/scripts/deploy.sh` for details.

---

## Project structure

- `frontend/` — React app, agent implementations, UI components
- `backend/` — LLM proxy (Lambda + API Gateway)
- `shared/` — TypeScript types and Zod schemas
- `scripts/create-agent.ts` — Agent scaffolding

See [frontend/src/agents/README.md](frontend/src/agents/README.md) for agent development conventions and the design system.

---

## Building an Agent

| Who you are | Start here |
|-------------|-----------|
| **Human developer or designer** | [DEVELOPER-GUIDE.md](DEVELOPER-GUIDE.md) — environment setup, local dev, build, deploy, and how to request an agent from a coding agent |
| **AI coding agent** | [AGENT-GUIDE.md](AGENT-GUIDE.md) — complete implementation reference: imports, file structure, LLM loop pattern, framework exports, UI conventions, and conformance checklist |
| **Designer requesting AI implementation** | [AGENT-REQUEST-TEMPLATE.md](AGENT-REQUEST-TEMPLATE.md) — fill-in prompt template to give to a coding agent |
