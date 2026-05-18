---
description: Run before opening a uga-ool PR (author self-check)
---

You are helping the author **before** they open a uga-ool pull request.

Follow the **author track** in skill `ool-pr-and-code-review` and `docs/cursor/pr-and-code-review.md`.

1. **Repo/path** — Per `upstream-reference-repos.mdc`: new course-file React apps must be under `UGA-Brightspace-React-Apps/apps/<name>/`. List changed paths from `git status` and flag edits under `UGA-Brightspace-React-Template` or other reference repos unless the user explicitly targeted them.
2. **Secrets / FERPA** — Scan the diff for `.env`, API keys, tokens, student names, or UGA IDs in committed files or comments. Per `ool-secrets-and-ferpa.mdc`.
3. **Build** — Run the build step from the app or repo README (e.g. `npm run build` in the app directory) when applicable; report pass/fail.
4. **Output** — Pass/fail checklist; suggested **Summary** and **Test plan** bullets for the PR description (include eLC OU/role/path when relevant).
5. **Do not** commit, push, or open the PR unless the user explicitly asks.

Address the user's repo or branch context.
