# UGA Online — Cursor team documentation

Central reference for **eLC (Brightspace)**, **OOL templates**, **AI agents**, **video and content tooling**, and related deployment notes. Add this repo to a multi-root workspace or clone it beside project repositories so the team shares one layout.

## Where everything lives

| Path | Purpose |
|------|---------|
| [`docs/INDEX.md`](docs/INDEX.md) | **Start here** — documentation grouped by **job** (D2L APIs, deploy, agents, React apps, Lit library, video, captions, PDF). |
| [`docs/`](docs/) | All markdown and HTML reference material, organized **by topic** (no per-repo mirror folder). |
| [`docs/_meta/manifest.md`](docs/_meta/manifest.md) | Which upstream repo each topic area came from, for updates. |
| [`.cursor/rules/`](.cursor/rules/) | Cursor rules (MDC): upstream reference preservation, OOL GitHub practices, Brightspace React course files, design system, eLC AI agent plugins. |
| [`.cursor/skills/`](.cursor/skills/) | Agent skills (e.g. scaffold from OOL template). |
| [`.cursor/commands/`](.cursor/commands/) | Short command prompts aligned with those rules. |

## Agent framework forks

Repos such as `agent-todd` and `thestater-agent` follow the same patterns as **`uga-online-brightspace-agent-framework-main`**. Canonical guides and example prompts for the team are under [`docs/ai-agents-brightspace/`](docs/ai-agents-brightspace/).

## Security

Do not commit API keys, OAuth secrets, or LLM keys. Keep credentials in gitignored env files or your institutional secret store.

## Keeping docs current

When upstream project docs change, update the matching files under `docs/<topic>/` using [`docs/_meta/manifest.md`](docs/_meta/manifest.md) as a map to source repositories.
