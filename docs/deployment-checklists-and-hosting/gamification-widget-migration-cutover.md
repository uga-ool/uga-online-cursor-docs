# Gamification Widget Agent Migration - QA and Cutover

## What was migrated
- Original standalone widget remains at `Vibe Coding Widget.html` for rollback.
- New framework-style agent implementation lives at:
  - `frontend/src/agents/gamification-widget-agent/config.ts`
  - `frontend/src/agents/gamification-widget-agent/entry.tsx`
  - `frontend/src/agents/gamification-widget-agent/entry.html`
  - `frontend/src/agents/gamification-widget-agent/components/MainView.tsx`
  - `frontend/src/agents/gamification-widget-agent/services/*`
  - `frontend/src/agents/gamification-widget-agent/prompts/*`
  - `frontend/src/agents/gamification-widget-agent/data/*`

## Validation executed
- Workspace dependency install:
  - `npm install` in this repo
  - `npm install` in `../uga-online-brightspace-agent-framework-main` for cross-repo framework imports
- Build checks:
  - `npm run build --workspace=frontend`
  - `npm run build:agent --workspace=frontend -- gamification-widget-agent`

## Feature parity checklist
- [x] Four-step authoring workflow preserved (runtime, artifact, structured authoring, review/export)
- [x] Structured instructional fields preserved
- [x] Accessibility checklist hard-gating preserved
- [x] Prompt composition preserved and externalized
- [x] Copy and download exports preserved
- [x] Framework-driven generation path added (`runFrameworkGeneration`)

## Framework compatibility checklist
- [x] Uses plugin manifest via `defineAgentPlugin`
- [x] Uses workflow builder in `config.ts`
- [x] Registers framework tools in `entry.tsx`
- [x] Uses `BrightspaceProvider` runtime wrapper
- [x] Uses stable `@uga-brightspace/framework` import surface

## Cutover plan
1. Build agent: `npm run build:agent --workspace=frontend -- gamification-widget-agent`
2. Upload generated assets to deployment target (course/shared hosting strategy).
3. Point launch entry to `frontend/src/agents/gamification-widget-agent/entry.html` equivalent deployed artifact.
4. Run smoke tests:
   - complete all required fields and confirm gating behavior
   - run generation and verify output appears
   - copy/download prompt/output
   - keyboard-only traversal and focus visibility
5. Sign off with faculty stakeholder on content quality and guardrail behavior.

## Rollback plan
- If production issues are found, revert launch to standalone `Vibe Coding Widget.html`.
- Keep migrated agent code intact for patching and re-deployment.
- Re-run build + smoke tests before next cutover attempt.
