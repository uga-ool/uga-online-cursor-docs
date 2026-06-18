# Example multi-root workspaces

**Not required.** These illustrate how to split folders so Agent mode is less likely to edit template clones. Adjust before use.

## How to use

1. Copy a `.code-workspace` file to your own folder (e.g. `~/local_dev/UGA-Online-Develop.code-workspace`).
2. Edit each `path` so it points at **your** clone locations. Paths in these examples assume:
   - You open the workspace file from `uga-online-cursor-docs/workspaces/examples/`, and
   - All `uga-ool` repos are siblings under the same parent (e.g. `local_dev/`).
3. Add or remove folders to match repos you actually have checked out.
4. In Cursor: **File → Open Workspace from File…**

## Role → workspace → first command

| Role | Workspace file | First command / doc |
|------|----------------|---------------------|
| Instructional designer | `UGA-Online-InstructionalDesign.code-workspace` | `html-cleanup` · [`docs/cursor/roles/instructional-designer.md`](../../docs/cursor/roles/instructional-designer.md) |
| QA / analytics | `UGA-Online-QA.code-workspace` | `elc-smoke-test` · [`docs/cursor/roles/qa-analytics.md`](../../docs/cursor/roles/qa-analytics.md) |
| AI developer (apps) | `UGA-Online-Develop.code-workspace` | `pre-commit-review` (in template repo) · [`docs/cursor/roles/ai-developer.md`](../../docs/cursor/roles/ai-developer.md) |
| AI developer (agents) | `UGA-Online-Agents.code-workspace` | `elc-ai-agent` · [`docs/cursor/roles/ai-developer.md`](../../docs/cursor/roles/ai-developer.md) |
| Sysadmin | `UGA-Online-Develop.code-workspace` or hub only | [`docs/cursor/team-rules-dashboard-setup.md`](../../docs/cursor/team-rules-dashboard-setup.md) |
| Read patterns (any role) | `UGA-Online-Reference.code-workspace` | Ask mode · hub [`docs/INDEX.md`](../../docs/INDEX.md) |

Full catalog: [`docs/cursor/shared-resources-catalog.md`](../../docs/cursor/shared-resources-catalog.md).

## Which example

| File | Folders (concept) | Use when |
|------|-------------------|----------|
| `UGA-Online-Develop.code-workspace` | Docs + app/work repos (no template) | Daily feature work with Agent |
| `UGA-Online-Reference.code-workspace` | Docs + template, framework, lit | Reading patterns — prefer Ask mode |
| `UGA-Online-InstructionalDesign.code-workspace` | Docs + pdf-accessibility + lit (reference) | ID HTML and accessibility work |
| `UGA-Online-QA.code-workspace` | Docs + deployment checklists + React Apps | QA smoke tests and PR review |
| `UGA-Online-Agents.code-workspace` | Docs + agent fork + framework (reference) | Agent plugin development |
| `UGA-Online-Full.code-workspace` | All listed repos | Cross-repo work only; use sparingly |

If you already use a personal workspace (e.g. `eLC Workspace.code-workspace` at your repo root), keep using that — the **ideas** in the table matter more than these filenames.
