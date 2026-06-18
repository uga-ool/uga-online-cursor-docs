# QA and analytics — Cursor quick-start

~15 min. For eLC smoke testing, course review, and PR test-plan validation.

## Your workspace

Copy [`UGA-Online-QA.code-workspace`](../../workspaces/examples/UGA-Online-QA.code-workspace) and fix folder paths. Includes hub docs plus a React app folder for smoke testing.

## Cursor modes

| Mode | Use for |
|------|---------|
| **Ask** | Generate test plans, review course structure, summarize PR diffs |
| **Agent** | Draft test-plan markdown, update checklist docs — review diffs |
| **Plan** | New QA workflow or rollout checklist before implementation |

## Key commands

| Command | When |
|---------|------|
| `elc-smoke-test` | Walk through smoke test checklist for a tool |
| `uga-online-pr-review` | Review a developer PR (reviewer track) |
| `uga-online-pre-pr-check` | Verify your own doc/checklist PR before opening |

Subagents: **`elc-course-reviewer`** (rubric-based course review), **`uga-online-html-reviewer`** (accessibility HTML pass).

## Key docs

- Smoke test checklist: [`docs/deployment-checklists-and-hosting/D2L_SMOKE_TEST_CHECKLIST.md`](../deployment-checklists-and-hosting/D2L_SMOKE_TEST_CHECKLIST.md)
- Deployment and rollout: [`docs/deployment-checklists-and-hosting/`](../deployment-checklists-and-hosting/)
- Course review prompt patterns: [`docs/ai-agents-brightspace/example-prompts/course-review/`](../ai-agents-brightspace/example-prompts/course-review/)
- PR review playbook: [`pr-and-code-review.md`](pr-and-code-review.md)
- Prompt starters: [`prompts/eLC-test-plan.md`](prompts/eLC-test-plan.md)

## PR test plan fields

Every student-facing PR should include in the description:

- Course/OU tested
- Role tested (instructor, student, admin)
- Manage Files or Content path
- Keyboard-only and focus-state checks completed

## FERPA reminder

Use test-sandbox courses. Do not paste real student data into chat or committed files.

---

**Open this workspace** → **Run `elc-smoke-test`** → **Read [`D2L_SMOKE_TEST_CHECKLIST.md`](../deployment-checklists-and-hosting/D2L_SMOKE_TEST_CHECKLIST.md)**
