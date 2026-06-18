# AI developer — Cursor quick-start

~15 min. For Brightspace agent plugins, course-file React apps, and Lit components.

## Your workspace

| Task | Workspace |
|------|-----------|
| Daily app/agent work | [`UGA-Online-Develop.code-workspace`](../../workspaces/examples/UGA-Online-Develop.code-workspace) |
| Agent plugin focus | [`UGA-Online-Agents.code-workspace`](../../workspaces/examples/UGA-Online-Agents.code-workspace) |
| Read-only patterns | [`UGA-Online-Reference.code-workspace`](../../workspaces/examples/UGA-Online-Reference.code-workspace) — prefer **Ask** mode |

Open **one repo folder** when learning: the repo you commit to (Lit, agent framework, or React Apps).

## Commit → PR flow

1. **Stage changes** in the template repo you edit.
2. Run **`pre-commit-review`** (in that repo) before every commit.
3. Run **`uga-online-pre-pr-check`** (hub command or skill) before opening the PR.
4. Request review; reviewers use **`uga-online-pr-review`**.

Cadence: [`commit-rhythm.md`](commit-rhythm.md) · Full playbook: [`pr-and-code-review.md`](pr-and-code-review.md)

## Key commands by repo

| Repo | Commands / skills |
|------|-------------------|
| Agent framework | `pre-commit-review`, `elc-ai-agent`, `agent-before-commit`, `elc-valence-api` |
| React Apps | `pre-commit-review`, `uga-react-template`, `apps-before-commit`, `scaffold-from-uga-online-template` skill |
| Lit | `pre-commit-review`, `lit-before-commit`, `elc-valence-api`, `kaltura-api` |

Subagent: **`elc-agent-scaffolder`** when building new agent plugins.

## Key docs

- Agent framework: [`docs/ai-agents-brightspace/guides/AGENT-GUIDE.md`](../ai-agents-brightspace/guides/AGENT-GUIDE.md), [`DEVELOPER-GUIDE.md`](../ai-agents-brightspace/guides/DEVELOPER-GUIDE.md)
- React monorepo: [`docs/course-file-react-apps/monorepo-overview.md`](../course-file-react-apps/monorepo-overview.md)
- Template registry: [`template-repos.md`](template-repos.md)
- Prompt starter: [`prompts/pr-description-from-diff.md`](prompts/pr-description-from-diff.md)

## Repo tiers (do not skip)

- **Do not** commit app features to `UGA-Brightspace-React-Template`
- **New** React apps → `UGA-Brightspace-React-Apps/apps/<kebab-case-name>/`
- Agent imports → `@uga-brightspace/framework` only

---

**Open `UGA-Online-Develop` workspace** → **Run `pre-commit-review` in your repo** → **Read [`AGENT-GUIDE.md`](../ai-agents-brightspace/guides/AGENT-GUIDE.md) or app README**
