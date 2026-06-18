# UGA Online — Cursor team documentation

Central reference for **eLC (Brightspace)**, **UGA Online templates**, **AI agents**, **video and content tooling**, and related deployment notes. Add this repo to a multi-root workspace or clone it beside project repositories so the team shares one layout.

## Where everything lives

| Path | Purpose |
|------|---------|
| [`docs/INDEX.md`](docs/INDEX.md) | **Start here** — documentation grouped by **job** (D2L APIs, deploy, agents, React apps, Lit library, video, captions, PDF). |
| [`docs/`](docs/) | All markdown and HTML reference material, organized **by topic** (no per-repo mirror folder). |
| [`docs/_meta/manifest.md`](docs/_meta/manifest.md) | Which upstream repo each topic area came from, for updates. |
| [`cursor-demo.html`](cursor-demo.html) | **Onboarding** — VS Code, Cursor, GitHub, role quick-starts, Cursor Teams (open in a browser). |
| [`docs/cursor/shared-resources-catalog.md`](docs/cursor/shared-resources-catalog.md) | **Cursor catalog** — Team Rules, hub commands, role mapping (start here for shared resources). |
| [`docs/cursor/`](docs/cursor/) | Cursor workflows (PR review, chat practices, prompt starters, pilot rollout). |
| [`.cursor/agents/`](.cursor/agents/) | Role subagents (HTML reviewer, course reviewer, PR reviewer, agent scaffolder). |
| [`workspaces/examples/`](workspaces/examples/) | Optional example multi-root workspaces (copy and adjust paths; not required). |
| [`.cursor/rules/`](.cursor/rules/) | Cursor rules (MDC): upstream reference, secrets/FERPA, UGA Online GitHub, Brightspace React, design system, eLC AI agents. |
| [`.cursor/skills/`](.cursor/skills/) | Agent skills (scaffold from template, PR and code review). |
| [`.cursor/commands/`](.cursor/commands/) | Commands including `uga-online-pre-pr-check`, `uga-online-pr-review`, and topic prompts. |

## Agent framework forks

Repos such as `agent-todd` and `thestater-agent` follow the same patterns as **`uga-online-brightspace-agent-framework-main`**. Canonical guides and example prompts for the team are under [`docs/ai-agents-brightspace/`](docs/ai-agents-brightspace/).

## Security

Do not commit API keys, OAuth secrets, or LLM keys. Keep credentials in gitignored env files or your institutional secret store.

After clone, run [`scripts/setup-git-hooks.sh`](scripts/setup-git-hooks.sh) so commits follow [`.github/COMMIT_TEMPLATE`](.github/COMMIT_TEMPLATE).

## Keeping docs current

When upstream project docs change, update the matching files under `docs/<topic>/` using [`docs/_meta/manifest.md`](docs/_meta/manifest.md) as a map to source repositories.
