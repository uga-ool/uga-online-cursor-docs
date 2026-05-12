# uga-drive-elc-sync

Local-first tooling for **Markdown → HTML**, **pairing/status reports**, and **pluggable deployment** toward eLC (Brightspace). Content transformations are **deterministic scripts only**—no LLM in the pipeline. **Markdown is the source of truth**; generated HTML is the deploy artifact. Do not treat HTML exported from eLC as a safe source for overwriting local files without review.

## Recommended workflow: Google Drive for Desktop (no Drive API)

**Easiest path:** use [Google Drive for Desktop](https://support.google.com/drive/answer/2374987) so your course folder exists as a **normal directory** on disk. Add that folder to your Cursor workspace (multi-root). This tool then reads and writes **local files only**—no Google Cloud project, service account, or `drive pull` required for day-to-day work.

1. Edit Markdown in the synced folder (or a subfolder you use for sources).
2. Point this project at that folder via [`config/content-map.json`](config/content-map.json) — see [Content roots](#content-roots).
3. Run `process` → `validate` → `deploy` (mock until a real Brightspace adapter exists).

**Native Google formats (Docs, Sheets, Slides, Drawings):** Drive for Desktop does **not** replace the Drive API for these files. They usually appear as shortcuts or web links, not as real PDF or Microsoft Office binaries on disk. To convert them automatically for eLC, use **`content-sync drive pull`** (see [Optional: CLI `drive pull`](#optional-cli-drive-pull-drive-rest-api)) with [`nativeExports` in `config/drive-sync.json`](config/drive-sync.example.json), or import via the **Brightspace web app** in [`frontend/`](frontend/), which uses the same `files.export` behavior. You can also use **File → Download** in the Google editors and sync the downloaded files like any other upload.

### Workshop example: “eLC Sync Test”

If you add a Drive-backed folder to the workspace named **eLC Sync Test**, set `markdownRoot` to that folder’s **absolute path** (from Finder: hold Option and copy path, or drag into Terminal). Keep `htmlRoot` under this repo (e.g. `content/html`) so generated HTML stays with the tool. Example:

```json
{
  "markdownRoot": "/Users/you/Library/CloudStorage/GoogleDrive-your@email/My Drive/eLC Sync Test",
  "htmlRoot": "content/html",
  "pairing": "mirror"
}
```

Adjust the path for your machine. Then map deploy output in [`config/elc-destinations.json`](config/elc-destinations.json), e.g. `localPrefix` `content/html` and `elcPath` `course_content/elc_sync_test` for Manage Files paths in manifests.

**Alternative:** symlink `content/markdown` → your `eLC Sync Test` folder and leave the default `content-map.json` relative paths.

### Optional: CLI `drive pull` (Drive REST API)

If you prefer pulling from Google’s servers without Drive for Desktop, use **`content-sync drive pull`** and [`config/drive-sync.json`](config/drive-sync.example.json). This is **optional**; the default story above does not need it.

---

## Requirements

- Node.js 20+
- npm (or pnpm/yarn)

## Setup

```bash
cd uga-drive-elc-sync
npm install
npm run build
```

Optional: set `CONTENT_SYNC_ROOT` to the project root when running from another directory.

## Web app (Brightspace): Drive ↔ Manage Files

The **React** UI for importing from Google Drive, comparing modified times, and exporting to Drive lives under [`frontend/`](frontend/). It is separate from the **`elc-import-from-google-drive`** repository so you can keep that project on a known-good version.

**`VITE_GOOGLE_CLIENT_ID`** must be set at **build** time (Vite inlines it). Without it, **Connect Google Drive** stays disabled. Use a Google OAuth **Web** client; add your eLC origin and `http://localhost:5173` under **Authorized JavaScript origins**. Full checklist, production `.env.production`, **Manage Files base path** (`/` for root), Manage Files **403** troubleshooting, and Brightspace OAuth scopes: **[`frontend/DEPLOYMENT.md`](frontend/DEPLOYMENT.md)**.

```bash
cd frontend
cp .env.example .env
# set VITE_GOOGLE_CLIENT_ID
npm install
npm run build
```

For release builds you can copy [`frontend/.env.production.example`](frontend/.env.production.example) to `frontend/.env.production` instead of passing env on the command line.

Deploy `frontend/dist/` (`index.html` + `assets/`) into course files (same-origin as Brightspace). **`npm run build`** in `frontend/` also produces **`frontend/elc-sync-course-files.zip`** (archive root = `index.html` + `assets/`) for Manage Files import. From the repo root you can run **`npm run build:all`** to build the CLI and the web app together.

Details: [`frontend/README.md`](frontend/README.md).

## Content roots

[`config/content-map.json`](config/content-map.example.json) defines:

- **`markdownRoot`** — where `.md` files are read from (relative to the project root, or an **absolute** path to a Drive-synced folder).
- **`htmlRoot`** — where generated `.html` is written (usually `content/html` under the project; can also be absolute if you need it).

Paths are resolved in [`src/config/loadConfig.ts`](src/config/loadConfig.ts) (`resolveContentPath`).

## Running in Cursor (or any terminal)

After `npm run build`:

```bash
npm run content-sync -- status
npm run content-sync -- process content/markdown/sample/intro.md
npm run content-sync -- process --all
npm run content-sync -- process --outdated
npm run content-sync -- validate
npm run content-sync -- deploy --files content/html/sample/intro.html --dry-run
npm run content-sync -- deploy --manifest reports/deploy-manifest.json
```

Development without compiling:

```bash
npm run content-sync:dev -- status
```

Global options:

- `-v` / `--verbose` — more log detail
- Per-command `--dry-run` where documented (e.g. `process`, `deploy`)

Every run appends a log file under `logs/`.

## Layout

| Path | Purpose |
|------|---------|
| `content/markdown/` | Default source Markdown (or use absolute `markdownRoot`) |
| `content/html/` | Generated HTML (gitignored by default) |
| `config/` | `content-map.json`, `elc-destinations.json`, `processing-rules.json`; optional `drive-sync.json` |
| `reports/` | `status.json`, optional `status.html`, deploy manifests, mock deploy output |
| `logs/` | Timestamped operation logs |
| `docs/` | Notes such as [future eLC deploy](docs/ELC_DEPLOY_FUTURE.md) |
| `reference/elc-import/` | **Reference-only** Valence Manage Files snippets for a future real deploy adapter |
| `src/` | TypeScript modules |

## Commands

| Command | Purpose |
|---------|---------|
| `status` | Pair Markdown/HTML, compare mtimes, write `reports/status.json`; `--verify` compares on-disk HTML to pipeline output; `--html` writes `reports/status.html` |
| `process` | Convert Markdown to cleaned HTML; one path, `--all`, or `--outdated` |
| `validate` | Walk configured HTML root, run cleanup + validation + eLC mutation heuristics |
| `deploy` | `--files` (comma-separated) or `--manifest`; mock adapter copies to `reports/mock-elc-out/` |
| `drive pull` | **Optional** — download a Drive folder via REST API; see [Optional Drive API](#optional-cli-drive-pull-drive-rest-api) |

## Configuration

- **`config/content-map.json`** — Markdown and HTML roots (`pairing`: `mirror` = same relative path, `.md` → `.html`). Absolute paths supported for `markdownRoot` / `htmlRoot`.
- **`config/elc-destinations.json`** — Maps local prefixes (e.g. `content/html`) to logical ELC Manage Files paths for deploy manifests.
- **`config/processing-rules.json`** — Cleanup toggles, strict plain-text check, validation behavior.

## Safety

- Pipeline checks that **plain text** from the Markdown render matches plain text after cleanup; otherwise processing **fails** (no silent wording changes).
- Optional `status --verify` detects on-disk HTML that does not match what the Markdown pipeline would produce.

## Optional CLI `drive pull` (Drive REST API)

Drive access uses the **same Drive REST v3 endpoints** as **elc-import-from-google-drive** (`https://www.googleapis.com/drive/v3/files`, `alt=media`, `/export`) — compare `frontend/src/lib/drive-api.ts` there. This CLI uses **`fetch` + `Authorization: Bearer`** ([`src/drive/driveRest.ts`](src/drive/driveRest.ts)).

### Authentication (pick one)

1. **OAuth access token** — Set **`GOOGLE_DRIVE_ACCESS_TOKEN`** (same scope as the import app, e.g. `drive.readonly`).
2. **Service account** — `GOOGLE_APPLICATION_CREDENTIALS` or `auth.keyFile` in `drive-sync.json`; share the Drive folder with the service account email.

### Commands

```bash
npm run content-sync -- drive pull --profile my_course
npm run content-sync -- drive pull --folder DRIVE_FOLDER_ID --local content/markdown/drive_import --dry-run
```

Copy `config/drive-sync.example.json` to **`config/drive-sync.json`** (gitignored) to use profiles.

## eLC / Brightspace deploy

- **Today:** mock deploy only; see [`docs/ELC_DEPLOY_FUTURE.md`](docs/ELC_DEPLOY_FUTURE.md) for what a real Brightspace Manage Files adapter would need.
- You can bridge with the **elc-import-from-google-drive** browser tool for uploads until a Node adapter exists.

## Tests

```bash
npm test
```

## License

Private / internal unless stated otherwise.
