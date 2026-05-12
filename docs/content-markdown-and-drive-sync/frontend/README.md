# Web UI (Brightspace): UGA Drive ↔ eLC Sync

Vite + React **Google Drive ↔ Manage Files** (import, compare freshness, export). Same-origin Valence calls as **`elc-import-from-google-drive`**, but this app lives in **`uga-drive-elc-sync/frontend`** so the other repo can stay on a fixed version.

**Import cannot be completed** until **Brightspace user permissions** allow **Manage Files** (list, create folders, upload) for the course. Without that, Valence returns **403**—see **[DEPLOYMENT.md](DEPLOYMENT.md)** (especially §3b). If **`GET .../managefiles/`** works but **`POST .../managefiles/folder`** does not, the role may be **list-only**; **create/upload** must be enabled. Wrong **org unit** (`?ou=`) also causes **403**—compare the **Course org unit (API)** line in the UI to the course.

## Google Cloud

1. Enable **Google Drive API**.
2. Create an OAuth **Web client** (not Desktop).
3. Under **Authorized JavaScript origins**, add every origin where the app is hosted (e.g. `https://your-elc.brightspace.com` and `http://localhost:5173` for local dev). The GIS token client requires **JavaScript origins** for the popup/token flow.
4. Scopes: default is **`https://www.googleapis.com/auth/drive.readonly`** (import + compare). **Export (eLC → Drive)** needs a write-capable scope, e.g. **`https://www.googleapis.com/auth/drive.file`** or **`https://www.googleapis.com/auth/drive`**. Set `VITE_GOOGLE_DRIVE_SCOPE` at build time and re-consent users after changing scopes.

**Pre-flight checklist:** Drive API enabled, consent screen / test users, and the **Web client ID** you pass as `VITE_GOOGLE_CLIENT_ID` matches the client whose JavaScript origins you configured. Full steps: **[DEPLOYMENT.md](DEPLOYMENT.md)**.

If the eLC page **Content-Security-Policy** blocks **`https://accounts.google.com`**, the Google Identity Services script will not load—adjust CSP to allow that script origin for this tool.

## Brightspace OAuth2 app (D2L)

Manage Files Valence routes need **LP** scopes (see [Valence scopes](https://docs.valence.desire2learn.com/http-scopestable.html)):

- **`managefiles:files:manage`** — upload + save files  
- **`managefiles:folders:manage`** — create folders  

Confirm **LP 1.49+** for Manage Files routes. Details: **[DEPLOYMENT.md](DEPLOYMENT.md)** §4.

## Manage Files base path

The UI field must use **Manage Files** paths (see the Manage Files tool in the course), **not** Brightspace Content URLs from the address bar. Use **`/`** for the **course root** (entire tree under Manage Files); use a subfolder name for a single branch (e.g. `drive_import`). Full detail: **[DEPLOYMENT.md](DEPLOYMENT.md)** §3a.

## Environment variables (build)

| Variable | Description |
|----------|-------------|
| `VITE_GOOGLE_CLIENT_ID` | OAuth 2.0 Web **Client ID** (public; safe in static JS). **Required** for Connect Google Drive. |
| `VITE_GOOGLE_DRIVE_SCOPE` | Optional; defaults to `drive.readonly`. Use `drive.file` or `drive` for **Export to Drive**. |
| `VITE_DRIVE_EXPORT_*` | Optional per-type MIME/extension for Google-native files when the UI preset is **Build defaults**. |
| `VITE_DEFAULT_EXPORT_PRESET` | Optional initial value for the **Export format** dropdown (`build_defaults` \| `pdf` \| `microsoft` \| `html_docs`). |

The **Export format** control (PDF, Microsoft, HTML-for-Docs, or build defaults) applies to **Google Docs, Sheets, Slides, and Drawings** on import and compare. See **[DEPLOYMENT.md](DEPLOYMENT.md)** §3a1.

Copy [`.env.example`](.env.example) to `.env` for local dev. For production builds, copy [`.env.production.example`](.env.production.example) to **`.env.production`** (set your client ID there) or pass `VITE_*` on the command line — see **[DEPLOYMENT.md](DEPLOYMENT.md)** §2.

```bash
cd frontend
VITE_GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com npm run build
```

Deploy **`dist/index.html`** and **`dist/assets/`** together (same relative paths).

After **`npm run build`**, **`elc-sync-course-files.zip`** is written next to **`dist/`**. Extract or upload that archive to **Manage Files** so **`index.html`** sits at the folder root (not nested under `dist/`).

## Google OAuth tokens vs `elc-import-from-google-drive`

Drive tokens are stored in `sessionStorage` under keys prefixed **`uga_drive_elc_sync_`**, so they do not collide with the other app if both run on the same origin.

## Local development

```bash
cd frontend
cp .env.example .env
# Edit .env: set VITE_GOOGLE_CLIENT_ID

npm install
npm run dev
```

Add `http://localhost:5173` to **Authorized JavaScript origins** for the Web client.

## iframe-resizer

The bundle includes **`iframe-resizer` contentWindow** so the host eLC page can resize the iframe. If the parent does not load the matching script, you may see a timeout warning in the console; the app can still run.
