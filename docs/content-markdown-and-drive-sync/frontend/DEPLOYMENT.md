# Deployment and verification (UGA Drive ↔ eLC Sync web app)

This guide matches **`elc-import-from-google-drive`**’s deployment story; the same Google GIS + Valence requirements apply. Paths are relative to the **`uga-drive-elc-sync`** repo root (`cd frontend` = this folder).

**Prerequisite:** End-to-end **import** (Drive → Manage Files) **cannot be completed** until **Brightspace grants the relevant user permissions** in the target course: roles must allow **Manage Files** usage (list, create folders, upload). Technical setup (Google client ID, build, deploy) alone is not enough if Valence denies **403**—fix roles / `?ou=` first; details in **§3b** below.

Follow this after [Authorized JavaScript origins](https://console.cloud.google.com/apis/credentials) are set for your OAuth **Web** client.

## 1. Google Cloud pre-flight (confirm)

- [ ] **Google Drive API** is enabled in the same project as the OAuth client.
- [ ] **OAuth consent screen** is configured. If the app is in **Testing**, add every Google account that will use the tool under **Test users**.
- [ ] **OAuth 2.0 Client ID** — use the **Web application** client’s ID (ends in `.apps.googleusercontent.com`) as `VITE_GOOGLE_CLIENT_ID` in the build. It must be the client whose **Authorized JavaScript origins** include:
  - your eLC origin (e.g. `https://ugatest2.view.usg.edu`), and
  - `http://localhost:5173` if you test locally with Vite.
- [ ] If the static app is hosted on a **different** origin than the main Brightspace URL, add that host’s origin too — GIS validates the **page origin** where the script runs.

## 2. Build

**Option A — one-shot (CI or shell):**

```bash
cd frontend
VITE_GOOGLE_CLIENT_ID=YOUR_CLIENT_ID.apps.googleusercontent.com npm run build
```

**Option B — local release build:** copy [`.env.production.example`](.env.production.example) to **`.env.production`**, set `VITE_GOOGLE_CLIENT_ID`, then:

```bash
cd frontend
npm run build
```

Vite reads `.env.production` automatically for `vite build`. Do not commit `.env.production` (it is gitignored).

Artifacts: `frontend/dist/` — deploy **`index.html`** and **`assets/`** together (same relative layout; Vite `base` is `./`).

**Env variable name:** use exactly **`VITE_GOOGLE_CLIENT_ID`** (underscores). Names like `VITE-GOOGLE-CLIENT` are wrong — Vite will not map them to `import.meta.env.VITE_GOOGLE_CLIENT_ID`, and the UI will stay in the “not configured” state until you fix the name and rebuild.

## 3. Deploy to eLC

- Upload **both** **`index.html`** and the **`assets/`** folder to the same Manage Files (or hosting) path. If `assets/index-*.js` 404s, the page stays blank. Keep `./assets/...` paths intact.
- Update the **Content** topic (or tool URL) to point at the **current** HTML file URL after each upload. A **`/d2l/error/.../404`** in the network tab usually means Brightspace could not load that topic’s file — wrong path, deleted file, or missing `assets/`.
- The app loads **iframe-resizer** `contentWindow` (bundled from the `iframe-resizer` npm package) so the host page’s iFrameSizer can measure height. Without it, the iframe often stays **collapsed** and looks like a blank white area even when the app loaded.
- Ensure the tool is opened in a **course context** so Brightspace exposes the course org unit to the app (see course detection in the app).

If deployment uses another hostname, add that origin under **Authorized JavaScript origins** in Google Cloud.

## 3a. Manage Files base path (import / compare / export)

The **Manage Files base path** field is the **Valence Manage Files** path, **not** the browser “Content” URL. Use:

| Value | Meaning |
|-------|---------|
| **`/`** | Course **Manage Files root** — list and sync the **entire** file tree under the course (can be slow on large courses). |
| **`drive_import`** or **`week1/materials`** | Only that folder under Manage Files (as in the Manage Files UI). |

**Do not** paste paths like **`/content/enforced4/3518105-CourseName/`** from the address bar — the Manage Files API uses virtual paths from the course root; those URLs return **404** and the tool will not list files. If you paste a Content URL that matches the usual pattern, the app **normalizes** it to the course-root-relative part (often empty), which is equivalent to using **`/`** for the whole tree.

## 3a1. Google-native export format (import / compare)

For **Google Docs, Sheets, Slides, and Google Drawings**, the tool uses Drive **`files.export`** with a MIME type. The **Export format** control in the UI sets that policy for the current browser session:

| Preset | Behavior |
|--------|----------|
| **Build defaults** | Uses optional **`VITE_DRIVE_EXPORT_*_MIME`** / **`_EXT`** variables from the **build** (same idea as CLI `nativeExports`). If unset, PDF for each native type. |
| **PDF** | All four kinds export as PDF (fixed; does not use per-type env for the export MIME). |
| **Microsoft Office** | Docs → `.docx`, Sheets → `.xlsx`, Slides → `.pptx`, drawings → PDF. |
| **HTML for Docs** | Docs → `.html`; Sheets, Slides, drawings → PDF. |

**`VITE_DEFAULT_EXPORT_PRESET`** (optional) sets the **initial** dropdown value when **`sessionStorage`** has no saved choice. Valid values: `build_defaults`, `pdf`, `microsoft`, `html_docs`.

**Batch / CLI:** `content-sync drive pull` still uses [`config/drive-sync.json`](../../config/drive-sync.example.json) **`nativeExports`**, not this UI.

## 3b. Manage Files 403 on import (blocks completion)

Until this is resolved, **the import workflow cannot be completed** for affected users—this is a **permission** requirement, not something the app can work around by configuration alone.

If **Connect Google Drive** works but **Import** fails with **`403 Forbidden`** on `/d2l/api/lp/.../managefiles/folder`, Brightspace is denying **Manage Files** writes for the **org unit ID** used in the API URL. Per Valence, that usually means no permission to **see the file listing or create and edit files** in that org unit—even if your **role** is allowed Manage Files in the course you expect, a **wrong `ou`** (common in embedded/iframed tools) produces the same 403.

- **Verify org unit:** Use the **Course org unit (API)** line in the tool UI. It must match the course offering’s org unit (check **Course Admin** or your admin). If it does not, append **`?ou=CORRECT_ORG_UNIT_ID`** to the tool’s URL and reload.
- **Course role:** In **Course Admin** → **Roles and Permissions**, ensure your role can use **Manage Files** (create/edit/upload) for that course.
- **Iframe / context:** Embedded pages may resolve a different org than the surrounding course; **`?ou=`** overrides detection.

**GET works, POST folder 403 (CSRF):** The app sends **`X-Csrf-Token`** on folder create, upload init, and save—same pattern as **uga-online-brightspace-agent-framework** (`GET /d2l/lp/auth/xsrf-tokens`, field **`referrerToken`**). Without that header, some tenants return **403** on mutating Manage Files calls even when **GET** listing succeeds.

**GET works, POST folder 403 (role):** If **`GET .../managefiles/`** returns **`200`** but **`POST .../managefiles/folder`** still returns **`403`** after CSRF is in place, Brightspace may still treat the role as **read/list only** for Valence writes. Course Admin → **Roles and Permissions** must allow **creating/editing files and folders**, not only viewing Manage Files.

## 4. Brightspace OAuth app (D2L) — confirm scopes

The Manage Files Valence calls require a Brightspace **OAuth 2.0** app (separate from Google). In **Manage Extensibility** → your app → **OAuth 2.0** scopes, confirm at least:

| Scope | Purpose |
|-------|---------|
| `managefiles:files:manage` | Upload and save files |
| `managefiles:folders:manage` | Create folders |

See [Valence scopes](https://docs.valence.desire2learn.com/http-scopestable.html) and LP **1.49+** for Manage Files routes.

## 5. Smoke test (manual)

Run in a **course** as instructor or instructional designer:

1. Open the deployed tool.
2. Confirm **Course org unit (API)** matches the course (use **`?ou=`** on the tool URL if the iframe shows the wrong org).
3. **Connect Google Drive** — complete account/consent; allow popups if the browser blocks them.
4. Enter a **Drive folder ID** (or `root`) and use **Preview file count** (Import tab).
5. Set **Manage Files base path** — e.g. **`/`** for the whole course tree, or **`drive_import`** for one folder. See **§3a**.
6. **Import** (optional): run **Import to Manage Files**, then in **Manage Files** confirm files under the base path.
7. **Compare** (optional): open **Compare freshness**, **Load comparison** — listing many folders can take a while; progress is shown while the course side is scanned.
8. **Export** (optional): set **`VITE_GOOGLE_DRIVE_SCOPE`** to a write-capable scope at build time, rebuild, reconnect Google, enter a **Drive folder ID (destination)**, then **Export Manage Files to Drive**.

**If Connect fails or GIS never loads:** the eLC page may use a strict **Content-Security-Policy**. Allow scripts from `https://accounts.google.com` (and typically `https://www.gstatic.com` if required by GIS) for the page hosting this tool. See [README.md](README.md) CSP note.

**If the iframe height looks wrong:** the bundle includes **iframe-resizer** `contentWindow`; a host `iFrameSizer` timeout in the console is often harmless if the page still works.

## 6. Optional: local dev

```bash
cd frontend
cp .env.example .env
# Set VITE_GOOGLE_CLIENT_ID in .env
npm install
npm run dev
```

Ensure `http://localhost:5173` is in **Authorized JavaScript origins**.
