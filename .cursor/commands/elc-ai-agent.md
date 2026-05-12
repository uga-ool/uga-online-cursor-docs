---
description: Implement or debug the custom AI Agent tool in eLC (Brightspace framework)
---

You are working on the **UGA Brightspace AI Agent** experience that runs **inside eLC (D2L Brightspace)**.

1. Treat the repo’s **AGENT-GUIDE.md** as authoritative for AI-authored code. For humans, defer to **DEVELOPER-GUIDE.md** and **ARCHITECTURE.md** when those exist in the same repository.
2. **Imports:** agent plugin code must use `import { ... } from '@uga-brightspace/framework'` only — no deep imports from `frontend/src/core/*`.
3. **Structure:** under `frontend/src/agents/<agent-id>/` keep `config.ts` (workflow + `defineAgentPlugin`), `entry.tsx`, `entry.html`, `components/MainView.tsx`, `prompts/`, and `data/tools.json` aligned with the canonical template agent (e.g. template-fidelity).
4. **LLM:** respect `VITE_LLM_MODE`, `VITE_BACKEND_URL`, and optional `VITE_BACKEND_STREAM_URL`. Never put production LLM API keys in client-side env vars that ship to the browser; backend uses `LLM_*` keys.
5. **Brightspace:** assume Valence calls need a real LMS session (OU, XSRF) unless the task is explicitly UI-only with mocks.
6. After edits, note which **eLC** test course or LTI placement should be used for verification.

Complete the user’s request with minimal, reviewable changes.
