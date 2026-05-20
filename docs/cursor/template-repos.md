# Template repos with Cursor config

Cursor **rules**, **commands**, **skills**, and **commit templates** live in each template repo — not in `uga-online-cursor-docs`.

## Registry

| Repository | Add new work | Commit template | Cursor guide |
|------------|--------------|-----------------|--------------|
| [uga-lit-components](https://github.com/uga-ool/uga-lit-components) | `src/components/uga-*.ts`, `demo/*.html` | `.github/COMMIT_TEMPLATE` | [docs/cursor/README.md](https://github.com/uga-ool/uga-lit-components/blob/main/docs/cursor/README.md) |
| [uga-online-brightspace-agent-framework](https://github.com/uga-ool/uga-online-brightspace-agent-framework) | `frontend/src/agents/<name>/` | `.github/COMMIT_TEMPLATE` | [docs/cursor/README.md](https://github.com/uga-ool/uga-online-brightspace-agent-framework/blob/main/docs/cursor/README.md) |
| [UGA-Brightspace-React-Apps](https://github.com/uga-ool/UGA-Brightspace-React-Apps) | `apps/<kebab-name>/` | `.github/COMMIT_TEMPLATE` | [docs/cursor/README.md](https://github.com/uga-ool/UGA-Brightspace-React-Apps/blob/main/docs/cursor/README.md) |

## Pre-commit review (each template repo)

After you **stage** files in Source Control, run command **`pre-commit-review`** in that repo’s Cursor window (Command Palette). It checks paths, secrets/FERPA, build, and suggests a one-line commit message.

| When | Command | Where |
|------|---------|--------|
| **Before each commit** | `pre-commit-review` | Template repo you are editing (Lit, agent framework, React Apps) |
| **Before opening a PR** | `ool-pre-pr-check` | `uga-online-cursor-docs` hub (or follow `ool-pr-and-code-review` skill) |

Skills `lit-before-commit`, `apps-before-commit`, and `agent-before-commit` contain the same checklist if you prefer to ask the agent by skill name.

**How often to commit, push, and open PRs:** [`commit-rhythm.md`](commit-rhythm.md). Each template repo’s `docs/cursor/how-to-commit.md` repeats the cadence table for developers who open only one repo folder.

## API knowledge (Cursor skills)

Each template repo includes [`docs/cursor/api-references.md`](https://github.com/uga-ool/uga-lit-components/blob/main/docs/cursor/api-references.md) (same path in each repo) and skills:

| Repo | D2L / Valence skill | Kaltura skill | Other |
|------|---------------------|---------------|--------|
| uga-lit-components | `elc-valence-api` | `kaltura-api` | — |
| uga-online-brightspace-agent-framework | `elc-valence-api` | (see lit repo for video) | LLM: `DEVELOPER-GUIDE.md` |
| UGA-Brightspace-React-Apps | `elc-valence-api` | `kaltura-api` | `timeline-aws-api` for `apps/timeline-js` |

Official docs: [Brightspace API reference](https://docs.valence.desire2learn.com/reference.html), [Kaltura API docs](https://developer.kaltura.com/api-docs/). Command **`api-help`** in each repo routes you to the right skill.

## Read-only reference (no bundled `.cursor/` here)

- **UGA-Brightspace-React-Template** — read patterns; new apps go under **UGA-Brightspace-React-Apps/apps/**.

## Onboarding

- Browser: [cursor-demo.html](../../cursor-demo.html)
- Optional topic index: [docs/INDEX.md](../INDEX.md)

## Maintainers

When updating shared novice docs (`how-to-commit.md`, `secrets.md` patterns), edit each template repo’s `docs/cursor/` and keep commit template format aligned.
