# Example multi-root workspaces

**Not required.** These illustrate how to split folders so Agent mode is less likely to edit template clones. Adjust before use.

## How to use

1. Copy a `.code-workspace` file to your own folder (e.g. `~/local_dev/OOL-Develop.code-workspace`).
2. Edit each `path` so it points at **your** clone locations. Paths in these examples assume:
   - You open the workspace file from `uga-online-cursor-docs/workspaces/examples/`, and
   - All `uga-ool` repos are siblings under the same parent (e.g. `local_dev/`).
3. Add or remove folders to match repos you actually have checked out.
4. In Cursor: **File → Open Workspace from File…**

## Which example

| File | Folders (concept) | Use when |
|------|-------------------|----------|
| `OOL-Develop.code-workspace` | Docs + app/work repos (no template) | Daily feature work with Agent |
| `OOL-Reference.code-workspace` | Docs + template, framework, lit | Reading patterns — prefer Ask mode |
| `OOL-Full.code-workspace` | All listed repos | Cross-repo work only; use sparingly |

If you already use a personal workspace (e.g. `eLC Workspace.code-workspace` at your repo root), keep using that — the **ideas** in the table matter more than these filenames.
