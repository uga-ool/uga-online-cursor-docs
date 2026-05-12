# Future: upload to Brightspace Manage Files (eLC)

Today, `content-sync deploy` uses a **mock** adapter that copies files to `reports/mock-elc-out/` and logs intent.

A **real** adapter should implement the same Valence LP behavior as the browser tool in **elc-import-from-google-drive**:

- `GET /d2l/lp/auth/xsrf-tokens` for `X-Csrf-Token`
- `POST .../managefiles/folder` to ensure folders
- Resumable upload + `POST .../managefiles/file/save`

Reference snapshots (not compiled into this CLI) live under [`reference/elc-import/`](../reference/README.md).

**Why it is not done here yet:** the import app uses **same-origin** `fetch` with the user’s eLC session. A Node/CLI client needs a **base URL**, **org unit id**, **LP API version**, and an **auth strategy** (OAuth bearer, service account if your campus supports it, or a small local helper that reuses a stored refresh token)—policies vary by institution.

When you implement `BrightspaceManageFilesDeployService`, wire it in [`src/deploy/deployService.ts`](../src/deploy/deployService.ts) and extend `DeployAdapterKind`.
