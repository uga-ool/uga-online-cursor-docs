# UGA Online shared Cursor resources catalog

**Start here** for the team's three-tier Cursor resource model. Deep technical reference remains in [`docs/INDEX.md`](../INDEX.md).

## Three tiers

| Tier | Where | What lives here |
|------|-------|-----------------|
| **1 — Cursor Teams dashboard** | [cursor.com/dashboard](https://cursor.com/dashboard) → Rules | Enforced org-wide Team Rules (FERPA, repo tiers, design system); optional glob-scoped rules |
| **2 — Hub repo** | `uga-online-cursor-docs` `.cursor/` + `docs/cursor/` | Cross-repo rules, skills, commands, subagents, onboarding, this catalog |
| **3 — Template / work repos** | Lit, agent framework, React Apps, etc. | Repo-scoped rules, `pre-commit-review`, API skills |

**Precedence:** Team Rules → Project Rules → User Rules.

Setup guide for tier 1: [`team-rules-dashboard-setup.md`](team-rules-dashboard-setup.md).

---

## Team Rules (Cursor dashboard)

Configure in **cursor.com/dashboard → Rules**. Canonical `.mdc` sources live in this hub for version control; paste into the dashboard when updating.

### Enforced (always apply)

| Rule name | Source file | Enforced |
|-----------|-------------|----------|
| `uga-online-secrets-and-ferpa` | [`.cursor/rules/uga-online-secrets-and-ferpa.mdc`](../../.cursor/rules/uga-online-secrets-and-ferpa.mdc) | Yes |
| `uga-online-upstream-repo-tiers` | [`.cursor/rules/upstream-reference-repos.mdc`](../../.cursor/rules/upstream-reference-repos.mdc) | Yes |
| `uga-online-design-system` | [`.cursor/rules/uga-online-design-system.mdc`](../../.cursor/rules/uga-online-design-system.mdc) | Yes |

### Optional (glob-scoped)

| Rule name | Glob | Source file |
|-----------|------|-------------|
| `elc-ai-agent-framework` | `**/frontend/src/agents/**` | [`.cursor/rules/elc-ai-agent-tool.mdc`](../../.cursor/rules/elc-ai-agent-tool.mdc) |
| `uga-react-course-files` | `**/src/**/*.{ts,tsx}` | [`.cursor/rules/uga-brightspace-react-template.mdc`](../../.cursor/rules/uga-brightspace-react-template.mdc) |
| `uga-online-html-cleanup` | `**/*.{html,htm}` | [`.cursor/rules/uga-online-html-cleanup.mdc`](../../.cursor/rules/uga-online-html-cleanup.mdc) |

---

## Hub project rules (`.cursor/rules/`)

| File | alwaysApply | Scope |
|------|-------------|-------|
| `uga-online-secrets-and-ferpa.mdc` | true | Secrets, FERPA, PR red flags |
| `upstream-reference-repos.mdc` | true | Repo tiers, where new apps live |
| `uga-online-design-system.mdc` | true | UGA Online Design System (always) |
| `uga-ool-github.mdc` | false | Org GitHub practices |
| `uga-brightspace-react-template.mdc` | false | `**/src/**/*.{tsx,ts}` |
| `elc-ai-agent-tool.mdc` | false | `**/frontend/src/agents/**` |
| `uga-online-html-cleanup.mdc` | false | `**/*.{html,htm}` — accessibility cleanup + required design system |

**Remote import:** Cursor Settings → Rules → Remote Rule (GitHub) → `https://github.com/uga-ool/uga-online-cursor-docs`.

---

## Hub skills (`.cursor/skills/`)

| Skill | Use when |
|-------|----------|
| [`uga-online-pr-and-code-review`](../../.cursor/skills/uga-online-pr-and-code-review/SKILL.md) | Author or reviewer PR checklist |
| [`scaffold-from-uga-online-template`](../../.cursor/skills/scaffold-from-uga-online-template/SKILL.md) | New course-file React app under `UGA-Brightspace-React-Apps/apps/` |

---

## Hub commands (`.cursor/commands/`)

| Command | Audience | Purpose |
|---------|----------|---------|
| `uga-online-pre-pr-check` | All authors | Self-check before opening a PR |
| `uga-online-pr-review` | Reviewers | Reviewer checklist |
| `elc-smoke-test` | QA, devs | Run eLC smoke test checklist |
| `html-cleanup` | IDs, QA | Apply HTML remediation + UGA Online Design System to a file |
| `agent-request` | IDs | Start an eLC agent feature request |
| `elc-ai-agent` | AI devs | Implement/debug agent plugins (entry point) |
| `elc-design-system` | Devs, IDs | Apply UGA Online Design System |
| `uga-react-template` | Devs | Scaffold/extend course-file React apps |
| `uga-ool-github` | All | Plan work across `uga-ool` repos |
| `setup-git-hooks` | All | Enable commit template hooks |

Palette: **⌘+Shift+P** (Mac) or **Ctrl+Shift+P** (Windows) — search the command name.

---

## Hub subagents (`.cursor/agents/`)

| Subagent | Primary users | Purpose |
|----------|---------------|---------|
| `uga-online-html-reviewer` | IDs, QA | Accessibility + semantic HTML review |
| `elc-course-reviewer` | QA, IDs | Rubric-based course structure review |
| `uga-online-pr-reviewer` | Devs | PR review via `uga-online-pr-and-code-review` skill |
| `elc-agent-scaffolder` | AI devs | Scaffold new Brightspace agent plugin |

---

## Template-repo commands and skills

Per [`template-repos.md`](template-repos.md). Open the **repo you commit to** for these.

| Repo | Pre-commit command | Before-commit skill | API skills |
|------|-------------------|---------------------|------------|
| `uga-lit-components` | `pre-commit-review` | `lit-before-commit` | `elc-valence-api`, `kaltura-api` |
| `uga-online-brightspace-agent-framework` | `pre-commit-review` | `agent-before-commit` | `elc-valence-api` |
| `UGA-Brightspace-React-Apps` | `pre-commit-review` | `apps-before-commit` | `elc-valence-api`, `kaltura-api`, `timeline-aws-api` |

Shared in all template repos: command `api-help`, commit template, `docs/cursor/how-to-commit.md`.

---

## Role → workspace → repo → command

| Role | Workspace example | Primary repo(s) | First command / doc |
|------|-------------------|-----------------|---------------------|
| **Instructional designer** | [`UGA-Online-InstructionalDesign.code-workspace`](../../workspaces/examples/UGA-Online-InstructionalDesign.code-workspace) | Hub + pdf-accessibility | `html-cleanup` · [roles/instructional-designer.md](roles/instructional-designer.md) |
| **QA / analytics** | [`UGA-Online-QA.code-workspace`](../../workspaces/examples/UGA-Online-QA.code-workspace) | Hub + React Apps (smoke test) | `elc-smoke-test` · [roles/qa-analytics.md](roles/qa-analytics.md) |
| **AI developer** | [`UGA-Online-Develop.code-workspace`](../../workspaces/examples/UGA-Online-Develop.code-workspace) or [`UGA-Online-Agents.code-workspace`](../../workspaces/examples/UGA-Online-Agents.code-workspace) | Agent framework / React Apps | `pre-commit-review` · [roles/ai-developer.md](roles/ai-developer.md) |
| **Sysadmin** | [`UGA-Online-Develop.code-workspace`](../../workspaces/examples/UGA-Online-Develop.code-workspace) | Hub + deploy repos | [roles/sysadmin.md](roles/sysadmin.md) |
| **PR author (any dev)** | Repo you commit to | Template repo | `pre-commit-review` → `uga-online-pre-pr-check` |
| **PR reviewer** | Hub or any repo | GitHub PR | `uga-online-pr-review` |

---

## Prompt libraries (do not conflate)

| Library | Location | Purpose |
|---------|----------|---------|
| **Cursor chat prompts** | [`docs/cursor/prompts/`](prompts/) | Starting messages, handoffs, review requests in Cursor IDE |
| **eLC in-course agent prompts** | [`docs/ai-agents-brightspace/example-prompts/`](../ai-agents-brightspace/example-prompts/) | LLM system prompts inside Brightspace agent tools |

Chat and transcript practices: [`chat-and-transcript-practices.md`](chat-and-transcript-practices.md).

---

## Workspaces (optional multi-root)

Copy from [`workspaces/examples/`](../../workspaces/examples/) and fix `path` entries. See [`workspaces/examples/README.md`](../../workspaces/examples/README.md).

| File | Use when |
|------|----------|
| `UGA-Online-Develop.code-workspace` | Daily app work — Agent mode |
| `UGA-Online-Reference.code-workspace` | Read templates/framework/lit — Ask mode |
| `UGA-Online-InstructionalDesign.code-workspace` | ID work — HTML, accessibility, design system |
| `UGA-Online-QA.code-workspace` | QA — smoke tests, deployment checklists |
| `UGA-Online-Agents.code-workspace` | Agent plugin development |
| `UGA-Online-Full.code-workspace` | All repos — use sparingly |

---

## Maintainer cadence

| When | Action | Owner |
|------|--------|-------|
| **Quarterly** | Compare Team Rules in dashboard to hub `.mdc` files; sync if drift | Coordinator |
| **On upstream change** | Refresh `docs/<topic>/` via [`docs/_meta/manifest.md`](../_meta/manifest.md) | Topic maintainer |
| **On new app/agent pattern** | Add repo-scoped rule or skill in template repo; link here | Repo maintainer |
| **After pilot feedback** | Update role quick-starts and this catalog | Coordinator + role leads |

**Owners:** Coordinator (hub + dashboard) + one maintainer per template repo (Lit, agent framework, React Apps).

---

## Related docs

- Onboarding: [`cursor-demo.html`](../../cursor-demo.html)
- Pilot rollout: [`pilot-rollout.md`](pilot-rollout.md)
- PR workflow: [`pr-and-code-review.md`](pr-and-code-review.md)
- Template registry: [`template-repos.md`](template-repos.md)
- Dashboard setup: [`team-rules-dashboard-setup.md`](team-rules-dashboard-setup.md)
- Documentation index: [`docs/INDEX.md`](../INDEX.md)
