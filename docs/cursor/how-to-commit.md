# How to commit (uga-online-cursor-docs hub)

This repo is mostly **documentation**. Commits still use [`.github/COMMIT_TEMPLATE`](../.github/COMMIT_TEMPLATE).

## One-time setup (after clone)

```bash
./scripts/setup-git-hooks.sh
```

Or Cursor command **`setup-git-hooks`**. Pull requests run the **Validate commit message** GitHub Action.

## Steps

1. Save files and stage changes in Source Control.
2. Run **`pre-commit-review`** if you use template-repo rules in this window (optional for doc-only edits).
3. Write **one imperative subject line** (10–72 characters). Delete all `#` template comment lines.
4. **Commit** → **Push** → open a **Pull request**.

Dry-run:

```bash
.github/scripts/validate-commit-message.sh --range origin/main..HEAD
```

## Cadence

See [commit-rhythm.md](commit-rhythm.md). Template repos hold app code; this hub holds team onboarding docs.
