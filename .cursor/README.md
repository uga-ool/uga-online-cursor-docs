# Cursor config in the UGA Online docs hub

This repo carries **org-wide** Cursor resources (tier 2). Template and work repos carry **repo-scoped** config (tier 3). Cursor Teams **dashboard Team Rules** (tier 1) apply automatically to all members.

## What lives here (hub)

| Path | Purpose |
|------|---------|
| [`.cursor/rules/`](rules/) | Cross-repo guardrails and conventions (FERPA, repo tiers, design system, HTML cleanup) |
| [`.cursor/skills/`](skills/) | Cross-repo procedures (PR review, scaffold from template) |
| [`.cursor/commands/`](commands/) | Workflow commands (`uga-online-pre-pr-check`, `elc-smoke-test`, `html-cleanup`, …) |
| [`.cursor/agents/`](agents/) | Role subagents (HTML reviewer, course reviewer, PR reviewer, agent scaffolder) |
| [`docs/cursor/shared-resources-catalog.md`](../docs/cursor/shared-resources-catalog.md) | **Canonical index** — start here |

## What lives in template repos

Open the repo you **commit to** for repo-scoped config:

| Repo | Cursor docs |
|------|-------------|
| [uga-lit-components](https://github.com/uga-ool/uga-lit-components) | `docs/cursor/README.md` |
| [uga-online-brightspace-agent-framework](https://github.com/uga-ool/uga-online-brightspace-agent-framework) | `docs/cursor/README.md` |
| [UGA-Brightspace-React-Apps](https://github.com/uga-ool/UGA-Brightspace-React-Apps) | `docs/cursor/README.md` |

Each template repo includes `pre-commit-review`, repo-specific skills (`elc-valence-api`, `*-before-commit`), and commit templates.

## How to use

1. **Join the UGA Online Cursor Teams org** — Team Rules apply automatically.
2. **Clone this hub** (and your work repo). Run [`scripts/setup-git-hooks.sh`](../scripts/setup-git-hooks.sh) once per clone.
3. **Open your work repo** in Cursor for daily commits; add this hub to a multi-root workspace or import remote rules if you need hub commands without multi-root.
4. **Read your role quick-start:** [`docs/cursor/roles/`](../docs/cursor/roles/).

Registry and role mapping: [`docs/cursor/shared-resources-catalog.md`](../docs/cursor/shared-resources-catalog.md).
