---
name: ool-pr-and-code-review
description: Help open or review GitHub pull requests in uga-ool repos. Use for code review, PR checklist, gh pr, drafting PR descriptions, or pre-PR author checks.
---

# OOL PR and code review

Full playbook: `docs/cursor/pr-and-code-review.md`

Respect `upstream-reference-repos.mdc`, `ool-secrets-and-ferpa.mdc`, and `uga-ool-github.mdc`.

## Author track

Use when the user is preparing to open a PR or asks for a pre-PR check.

1. **Repo/path** — New course-file React apps must be under `UGA-Brightspace-React-Apps/apps/<kebab-case>/`. Flag edits under `UGA-Brightspace-React-Template` or unintended reference repos.
2. **Pre-PR** — Prefer running the same steps as command `ool-pre-pr-check`: `git status`, scan diff for secrets and FERPA (student names/IDs), run `npm run build` (or README build step) when applicable.
3. **Draft PR body** — From `git diff main...HEAD` (or default branch): Summary (bullets), Test plan (checkboxes), eLC notes (OU, role, Manage Files path) if LMS-related.
4. **Do not** push, commit, or open the PR on GitHub unless the user explicitly asks.

## Reviewer track

Use when the user reviews someone else's PR.

1. **Context** — If PR URL or number given, use `gh pr view` and `gh pr diff` (read-only) when `gh` is available.
2. **Checklist**
   - Scope matches description
   - No secrets; no student PII in diff
   - No app-feature edits in template clone
   - eLC test plan present for student/instructor-facing changes
   - Monorepo apps under `apps/`; `VITE_*` only on client; design system for UI; agent imports from `@uga-brightspace/framework` only
3. **Output** — Short summary; **blocking** vs **non-blocking** findings; draft GitHub review comments (kind, specific).
4. **Ask mode preferred** — Do not merge, push, or rewrite author code unless the user explicitly requests fixes.

## Commands

- Author self-check: `ool-pre-pr-check`
- Reviewer: `ool-pr-review`
