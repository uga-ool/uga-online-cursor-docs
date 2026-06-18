# Pull requests and code review (UGA Online + Cursor)

Guide for teammates who are **new to GitHub PRs** or using **Cursor** on UGA Online (`uga-ool`) repositories.

## Glossary

| Term | Meaning |
|------|---------|
| **Branch** | A line of work separate from `main`; your commits live here until merged. |
| **PR (pull request)** | A request to merge your branch into `main`; where reviewers comment before merge. |
| **Diff** | The line-by-line comparison between your branch and `main`. |
| **Review** | A teammate reads the PR and approves or requests changes. |
| **Merge** | Accepting the PR so changes land on `main` (follow your repo’s policy). |

## Suggested cadence

Team defaults (guidance only—not enforced). Full detail: [`commit-rhythm.md`](commit-rhythm.md).

| Activity | Suggested cadence |
|----------|-------------------|
| Local commits | Every 30–90 minutes of meaningful progress (run **`pre-commit-review`** in the template repo each time) |
| Push to remote branch | 1–3 times per day |
| Pull request | Every 1–2 days |
| Merge to main | Daily or every few days (follow your lead and reviewers) |

## Which Cursor workspace to open

Optional **example** workspace files live in [`workspaces/examples/`](../../workspaces/examples/). Copy them to your machine and fix `path` entries — not required. If you already use a personal workspace (e.g. at your repo root), keep that.

| Workspace | Use when |
|-----------|----------|
| **UGA-Online-Develop** | Daily app work — Agent mode; no template repo in the tree |
| **UGA-Online-Reference** | Reading templates, framework, or lit library — prefer **Ask** mode |
| **UGA-Online-Full** | All repos at once — use sparingly; easy to edit the wrong clone |

See also [`upstream-reference-repos.mdc`](../../.cursor/rules/upstream-reference-repos.mdc).

## Before you open a PR

1. **Pre-commit on each commit** — In the **template repo** you edited, stage files and run **`pre-commit-review`** before every commit (see that repo’s `docs/cursor/how-to-commit.md`). Commits on your branch should already have passed path, secrets, and build checks.
2. **Correct repo** — New course-file React apps go under `UGA-Brightspace-React-Apps/apps/<kebab-case-name>/`. Do not commit feature work to `UGA-Brightspace-React-Template`.
3. **Small scope** — One feature or fix per PR when possible.
4. **`git status`** — No `.env` or API keys staged; no accidental edits in reference repos.
5. **Local checks** — Run what the app README says (usually `npm run build` from the app folder) if not already run via `pre-commit-review`.
6. **Pre-PR command** — In the **hub** repo or any window with hub rules, run **`uga-online-pre-pr-check`** (or ask the agent to follow the `uga-online-pr-and-code-review` skill author track) to draft the PR description and re-scan the branch diff.
7. **Commit messages** — Each commit must follow [`.github/COMMIT_TEMPLATE`](../../.github/COMMIT_TEMPLATE) (one imperative subject, no template comment lines left in). Run `./scripts/setup-git-hooks.sh` locally; PRs run the **Validate commit message** workflow.

## Branch protection (org admins)

On each `uga-ool` template repo, after the workflow is merged to `main`:

1. **Settings → Branches →** branch protection rule for `main`.
2. Enable **Require status checks to pass before merging**.
3. Select **Validate commit messages** (job name from [`.github/workflows/validate-commit-message.yml`](../../.github/workflows/validate-commit-message.yml)).
4. Save.

Authors with failing checks must fix messages (`git rebase -i` or squash merge with a clean subject). See [`template-repos.md`](template-repos.md).

## Opening a PR (author checklist)

1. Create a branch from `main` (e.g. `feature/short-description` or `fix/issue-topic`).
2. Commit with a clear message (what changed and why).
3. Push the branch to GitHub (`uga-ool` org).
4. Open a **pull request** on GitHub against `main`.
5. Fill in the description (template below).
6. Request one or more reviewers.
7. Address review comments on the **same branch** and push updates.

### PR description template

```markdown
## Summary
- 
- 

## Test plan
- [ ] Built locally (`npm run build` or per README)
- [ ] Tested in eLC (course/OU: ___; role: ___; Manage Files path: ___)
- [ ] 

## Notes
<!-- screenshots, tickets, intentional diffs from template -->
```

For **student-facing** or **instructor** tools, always include eLC validation details (OU, role, where the file is hosted).

### Using Cursor as author

- **Plan mode** — Large or risky tasks: agree on steps before Agent edits files.
- **Agent mode** — Implementation; **review every diff** before Accept (AI can be wrong).
- **Ask mode** — Draft PR summary from your changes: e.g. “Summarize `git diff main...HEAD` for a PR description.”
- **Never** commit `.env`, tokens, or student identifiers (see [`uga-online-secrets-and-ferpa.mdc`](../../.cursor/rules/uga-online-secrets-and-ferpa.mdc)).
- Do not ask Agent to push or open the PR unless you intend to — review `uga-online-pre-pr-check` output first.

## Reviewing a PR (reviewer checklist)

1. Read the **title**, **description**, and **test plan** on GitHub.
2. Open **Files changed** — Does the diff match the stated scope?
3. **Red flags**
   - Secrets (`.env`, API keys, tokens in source)
   - Edits under `UGA-Brightspace-React-Template` for app features
   - Unrelated refactors (“drive-by” changes)
   - Missing eLC test plan for LMS-facing UI
4. **UGA Online checks**
   - New React apps under `UGA-Brightspace-React-Apps/apps/`
   - Client env vars are `VITE_*` only; no secrets in bundles
   - UI uses [UGA Online Design System](https://design.online.uga.edu/getting-started/installation/) where applicable
   - Agent plugins import only from `@uga-brightspace/framework` (not deep paths)
5. **Optional local test** — `gh pr checkout <number>` then run the author’s test plan.
6. Leave **specific, kind** comments; **Approve** or **Request changes**.
7. **Merge** only if your team policy allows you to (many teams: author or lead merges after approval).

### Using Cursor as reviewer

- Prefer **Ask mode** (read-only) when exploring unfamiliar code.
- Run the **`uga-online-pr-review`** command and paste the PR URL or number.
- Sample prompts:
  - “Summarize this PR and risks for eLC deployment.”
  - “Does this diff introduce secrets or hard-coded org unit IDs?”
- Do not merge or push from Agent unless the reviewer explicitly asks.

## After review

- Author fixes on the same branch → push → re-request review on GitHub.
- Resolve conversations when addressed.
- Merge when approved (squash vs merge commit — follow repo maintainer preference).

## When to escalate

| Situation | Action |
|-----------|--------|
| PR is very large | Ask author to split into smaller PRs |
| Fix belongs in template/framework only | Separate upstream PR; don’t patch template clone for app work |
| Unclear eLC impact | Request test plan before approve |

## Team policy (outside Cursor)

Rules and skills **do not block** bad commits. Stronger safeguards (when your org enables them):

- Required PR reviews on `main`
- CI running `npm run build` on PRs
- GitHub secret scanning
- Repo `.github/pull_request_template.md` mirroring the template above

## Related links

- [Cursor demo (modes, diffs, FERPA)](../../cursor-demo.html)
- [`.cursor/rules/uga-ool-github.mdc`](../../.cursor/rules/uga-ool-github.mdc)
- [`.cursor/skills/uga-online-pr-and-code-review/`](../../.cursor/skills/uga-online-pr-and-code-review/)
- [uga-ool repositories](https://github.com/orgs/uga-ool/repositories)
