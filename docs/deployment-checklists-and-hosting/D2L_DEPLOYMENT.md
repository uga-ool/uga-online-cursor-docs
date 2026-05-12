# D2L Deployment Guide - Gamification Widget Agent

## 1) Build and package

From repo root:

```bash
npm run build:agent --workspace=frontend -- gamification-widget-agent
```

This now produces:
- `frontend/dist_d2l/gamification-widget-agent/`
  - `gamification-widget-agent.html` (deployable launcher)
  - `assets/` (compiled bundles)
- `frontend/dist_d2l/gamification-widget-agent-d2l.zip`

## 2) Configure runtime env before production build

Frontend `.env` values (copy from `frontend/.env.example`):
- `VITE_LLM_MODE=proxy`
- `VITE_BACKEND_URL=https://<backend-host>`
- optional `VITE_BACKEND_STREAM_URL=https://<stream-host>/api/llm/chat/stream`

Backend `.env` values (copy from `backend/.env.example`):
- provider defaults (`LLM_PROVIDER`, `LLM_MODEL`)
- provider API keys (`LLM_OPENAI_API_KEY`, etc.)

## 3) Upload to D2L

### Colocated (recommended first)
1. Upload `gamification-widget-agent.html` and full `assets/` folder to the same D2L Course Files folder.
2. Keep folder structure exactly:
   - `.../gamification-widget-agent.html`
   - `.../assets/*`

### Shared assets (optional)
1. Upload `assets/` to shared path.
2. Set `<meta name="app-shared-base" content="/shared/.../">` in deployed HTML.
3. Keep the HTML in course files and point to shared asset path.

## 4) Create/update External Learning Tool (LTI)

In D2L Admin:
1. Open External Learning Tools.
2. Create or edit tool for this app.
3. Set launch URL to deployed HTML file URL.
4. Enable in desired org units/courses.
5. Confirm iframe embedding policy permits your app domain and backend domain.

## 5) Production smoke tests

### Functional
- Complete all required fields -> generation should unlock.
- Run generation -> output appears.
- Copy prompt/output and download prompt/output both work.

### Accessibility
- Keyboard-only traversal works end-to-end.
- Focus visibility is clear on all controls.
- Status messages are announced in the UI.

### Integration
- UGA design system CSS/JS loads.
- Proxy calls to `/api/llm/chat` (and stream if used) succeed.

## 6) Cutover and rollback

### Cutover
- Set this tool as default entry for target courses after smoke test sign-off.

### Rollback
- Repoint launch to legacy standalone file:
  - `Vibe Coding Widget.html`
- Keep both deployment artifacts available until post-cutover validation window closes.
