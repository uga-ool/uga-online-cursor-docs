# Documentation index (by job)

Use this page first. All material lives under topic folders in [`docs/`](.)—there is no separate `sources/` tree.

## eLC / D2L platform and APIs

- **Valence, auth, OAuth, scopes:** [`d2l-brightspace/`](d2l-brightspace/)
- **Quizzes, dropbox, JSON formats:** [`d2l-brightspace/QUIZ_JSON_FORMAT.md`](d2l-brightspace/QUIZ_JSON_FORMAT.md), [`d2l-brightspace/D2L_QUIZ_DROPBOX.md`](d2l-brightspace/D2L_QUIZ_DROPBOX.md)
- **Course template / API spike:** [`d2l-brightspace/COURSE_TEMPLATE_API_SPIKE.md`](d2l-brightspace/COURSE_TEMPLATE_API_SPIKE.md)
## Markdown, Drive, and course files

- **Google ↔ eLC sync (specs and widget notes):** [`content-markdown-and-drive-sync/ELC_GOOGLE_SYNC_SPEC.md`](content-markdown-and-drive-sync/ELC_GOOGLE_SYNC_SPEC.md), [`content-markdown-and-drive-sync/ELC_GOOGLE_SYNC_WIDGET.md`](content-markdown-and-drive-sync/ELC_GOOGLE_SYNC_WIDGET.md)
- **Drive → eLC sync tool (overview, deploy, samples):** [`content-markdown-and-drive-sync/drive-elc-sync-overview.md`](content-markdown-and-drive-sync/drive-elc-sync-overview.md), [`content-markdown-and-drive-sync/frontend/`](content-markdown-and-drive-sync/frontend/), [`content-markdown-and-drive-sync/samples/`](content-markdown-and-drive-sync/samples/)

## Ship and harden (deploy, LTI, smoke tests)

- **Gamification LTI / D2L deploy / smoke / migration:** [`deployment-checklists-and-hosting/`](deployment-checklists-and-hosting/)
- **Timeline on AWS (SAM, rollout, hardening):** same folder plus [`timeline-aws/`](timeline-aws/) for API and data design
## AI agents in Brightspace

- **Framework guides:** [`ai-agents-brightspace/guides/AGENT-GUIDE.md`](ai-agents-brightspace/guides/AGENT-GUIDE.md), [`ai-agents-brightspace/guides/DEVELOPER-GUIDE.md`](ai-agents-brightspace/guides/DEVELOPER-GUIDE.md), [`ai-agents-brightspace/guides/ARCHITECTURE.md`](ai-agents-brightspace/guides/ARCHITECTURE.md)
- **Planning / backlog (TODO series):** [`ai-agents-brightspace/planning/`](ai-agents-brightspace/planning/)
- **Example agent prompts:** [`ai-agents-brightspace/example-prompts/`](ai-agents-brightspace/example-prompts/)
- **Data pipelines (market research):** [`ai-agents-brightspace/pipelines/`](ai-agents-brightspace/pipelines/)
- **Gamification agent prompts (product-specific):** [`lti-widgets-gamification/prompts/`](lti-widgets-gamification/prompts/)

## Course-file React apps (monorepo)

- **Monorepo and templates:** [`course-file-react-apps/monorepo-overview.md`](course-file-react-apps/monorepo-overview.md), [`course-file-react-apps/templates/`](course-file-react-apps/templates/)
- **Per-app READMEs:** [`course-file-react-apps/apps/`](course-file-react-apps/apps/)

## Lit component library

- **Overview, changelog, planning docs:** [`lit-component-library/overview.md`](lit-component-library/overview.md), [`lit-component-library/CHANGELOG.md`](lit-component-library/CHANGELOG.md)
- **Demo quick start, templates, servers, Copilot:** [`lit-component-library/demo/`](lit-component-library/demo/), [`lit-component-library/templates/`](lit-component-library/templates/), [`lit-component-library/server/`](lit-component-library/server/), [`lit-component-library/editor-copilot/`](lit-component-library/editor-copilot/)

## Video, timeline, captions

- **Kaltura video embed (`uga-video`):** [`lit-component-library/overview.md`](lit-component-library/overview.md) (Kaltura section)
- **Timeline AWS backend:** [`timeline-aws/`](timeline-aws/)
- **Captions / transcripts → course files:** [`captions-and-transcripts/`](captions-and-transcripts/)

## Accessibility

- **PDF remediation (designer HTML):** [`pdf-accessibility/`](pdf-accessibility/)

## Cursor editor (team resources)

- **Shared resources catalog (start here):** [`cursor/shared-resources-catalog.md`](cursor/shared-resources-catalog.md)
- **Role quick-starts:** [`cursor/roles/`](cursor/roles/) — instructional designer, QA, AI developer, sysadmin
- **Chat and transcript practices:** [`cursor/chat-and-transcript-practices.md`](cursor/chat-and-transcript-practices.md)
- **Cursor chat prompt starters:** [`cursor/prompts/`](cursor/prompts/)
- **Team Rules dashboard setup (admins):** [`cursor/team-rules-dashboard-setup.md`](cursor/team-rules-dashboard-setup.md)
- **Starter guide (HTML):** [`cursor-demo.html`](../cursor-demo.html) — VS Code, Cursor, GitHub, and this repo for newcomers
- **PR and code review (newcomers):** [`cursor/pr-and-code-review.md`](cursor/pr-and-code-review.md)
- **Commit / push / PR cadence (suggested):** [`cursor/commit-rhythm.md`](cursor/commit-rhythm.md)
- **Commit template and hooks:** [`cursor/how-to-commit.md`](cursor/how-to-commit.md), [`cursor/template-repos.md`](cursor/template-repos.md)
- **Workspace examples (optional, copy and adjust paths):** [`workspaces/examples/`](../workspaces/examples/)
- **Hub rules and commands:** [`.cursor/rules/`](../.cursor/rules/), [`.cursor/commands/`](../.cursor/commands/), [`.cursor/agents/`](../.cursor/agents/)
- **Always on:** [`upstream-reference-repos.mdc`](../.cursor/rules/upstream-reference-repos.mdc), [`uga-online-secrets-and-ferpa.mdc`](../.cursor/rules/uga-online-secrets-and-ferpa.mdc)
- **Skills:** [`scaffold-from-uga-online-template`](../.cursor/skills/scaffold-from-uga-online-template/), [`uga-online-pr-and-code-review`](../.cursor/skills/uga-online-pr-and-code-review/)
- **Commands:** `uga-online-pre-pr-check`, `uga-online-pr-review`, `elc-smoke-test`, `html-cleanup`, `agent-request` (see `.cursor/commands/`)

## Provenance and refreshing

- **Where each file came from (upstream paths):** [`_meta/manifest.md`](_meta/manifest.md)
