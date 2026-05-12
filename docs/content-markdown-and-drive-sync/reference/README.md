# Reference copies (not compiled)

The files under `elc-import/` are **snapshots** from [elc-import-from-google-drive](https://github.com/) for documentation only. They are **not** imported by the `content-sync` CLI.

Use them when implementing a future **Brightspace Manage Files** deploy adapter (Valence LP: folder create, resumable upload, save, XSRF). The browser app uses same-origin `fetch`; a Node adapter will need a base URL, OAuth bearer token or equivalent, and CSRF handling per your deployment.

**Drive:** The sibling repo lists/downloads via **Drive REST v3** + GIS OAuth in the browser (`drive-api.ts`). **uga-drive-elc-sync** mirrors those HTTP calls in [`src/drive/driveRest.ts`](file:///Users/todd/local_dev/uga-drive-elc-sync/src/drive/driveRest.ts) (Bearer token from `GOOGLE_DRIVE_ACCESS_TOKEN` or a service account).
