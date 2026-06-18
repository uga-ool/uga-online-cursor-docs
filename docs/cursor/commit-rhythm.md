# Suggested commit rhythm (UGA Online)

Team guidance for **how often** to commit, push, open pull requests, and merge. These are **suggestions** for developers and leads—not enforced by Git hooks or CI.

Adjust with your lead if your project or role needs a different pace.

## Cadence at a glance

| Activity | Suggested cadence |
|----------|-------------------|
| **Local commits** | Every **30–90 minutes** of meaningful progress |
| **Push to remote branch** | **1–3 times per day** |
| **Pull request (PR)** | Every **1–2 days** |
| **Merge to main** | **Daily** or every few days (when reviewers are available) |

## What each step means

### Local commits (30–90 minutes)

Commit when you finish a **logical chunk** of work: a component stub, a fixed demo, a workflow step, or a passing build—not every single line. Run **`pre-commit-review`** after staging and before each commit. Run **`./scripts/setup-git-hooks.sh`** once per clone so messages follow [`.github/COMMIT_TEMPLATE`](../.github/COMMIT_TEMPLATE) (see [`template-repos.md`](template-repos.md)).

Frequent local commits give you checkpoints on your branch and make diffs easier to review.

### Push to remote (1–3 times per day)

**Push** uploads your branch to GitHub so work is backed up and teammates can see progress. Push at least before you leave for the day; more often is fine on active feature branches.

### Pull request (every 1–2 days)

Open a **PR** when you have something reviewable—even if the feature is not finished. Small, frequent PRs are easier to review than one huge diff at the end of the week.

Before opening a PR, run hub command **`uga-online-pre-pr-check`** (or follow the [`uga-online-pr-and-code-review`](uga-online-pr-and-code-review.md) skill). See [`pr-and-code-review.md`](pr-and-code-review.md).

### Merge to main (daily or every few days)

**Merge** timing depends on reviewers and release policy. Aim to land reviewed PRs regularly so `main` stays current; your lead sets the exact rule for your team.

## Related docs

- Pre-commit checks: [`template-repos.md`](template-repos.md) — command `pre-commit-review`
- PR checklist: [`pr-and-code-review.md`](pr-and-code-review.md)
- Onboarding: [`cursor-demo.html`](../../cursor-demo.html)
