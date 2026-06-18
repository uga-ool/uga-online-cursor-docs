# UGA Online Cursor Teams pilot rollout

Two-week pilot before full team rollout. Coordinator runs this; update [`shared-resources-catalog.md`](shared-resources-catalog.md) after feedback.

## Pilot cohort (suggested)

| Role | Count | Participants (fill in) |
|------|-------|------------------------|
| Instructional designer | 2 | |
| QA / analytics | 1 | |
| AI developer | 2 | |
| Sysadmin | 1 | (Team Rules dashboard setup) |

## Week 1 — Setup and first tasks

**All participants**

1. Join UGA Online Cursor Teams org (invite link from coordinator).
2. Clone `uga-online-cursor-docs`; run `./scripts/setup-git-hooks.sh`.
3. Copy role workspace from [`workspaces/examples/`](../../workspaces/examples/).
4. Read role quick-start: [`docs/cursor/roles/`](roles/).

**Sysadmin**

1. Configure Team Rules per [`team-rules-dashboard-setup.md`](team-rules-dashboard-setup.md).
2. Verify Privacy Mode and invite links.

**Supervised first task (by role)**

| Role | Task | Success criteria |
|------|------|------------------|
| ID | Run `html-cleanup` on one sample HTML file | Corrected HTML; review summary if needed |
| QA | Run `elc-smoke-test` for one tool | Completed checklist in PR or doc |
| AI dev | `pre-commit-review` → commit → `uga-online-pre-pr-check` | PR description with eLC test plan |
| Sysadmin | Dashboard rules visible to pilot user | Enforced rules cannot be disabled |

## Week 2 — Handoffs and feedback

1. Run one **cross-role handoff** (ID → dev or dev → QA) using [`chat-and-transcript-practices.md`](chat-and-transcript-practices.md).
2. Collect friction log (shared doc or issue):

   - Which commands were hard to find?
   - Did Team Rules + project rules conflict?
   - Was workspace layout correct for the role?
   - Any FERPA or secrets near-misses?

3. Coordinator updates catalog and role quick-starts from feedback.

## Full rollout criteria

- [ ] Pilot cohort completed supervised tasks
- [ ] Team Rules synced with hub `.mdc` files
- [ ] No unresolved blocking feedback on catalog or onboarding
- [ ] `cursor-demo.html` reviewed by at least one non-developer

## Metrics (qualitative, first quarter)

- PRs include eLC test plan fields consistently
- Reviewers use `uga-online-pr-review`
- IDs use handoff templates vs long chat pastes
- Zero confirmed secrets/PII in committed files from pilot period

Onboarding entry point: [`cursor-demo.html`](../../cursor-demo.html).
