# UGA eLC — Kaltura transcripts → course Files

Batch-import **caption files (SRT/VTT)** from Kaltura entries referenced in a Brightspace (eLC) course into the course **Files** area (`transcripts/`), using:

1. **`frontend/`** — Vite + React app hosted in course Files (runs in the instructor’s eLC session). Env is **`VITE_*` only** (including **`VITE_N8N_IMPORT_WEBHOOK_URL`**).
2. **n8n workflow** — Receives webhook requests, calls Kaltura APIs (`session.start`, `caption_captionasset.list`, `caption_captionasset.serve`), then uploads caption files to Brightspace course Files using OAuth2 credentials.

[`UGA-Brightspace-React-Template-main/`](UGA-Brightspace-React-Template-main) is a **frozen snapshot** of the UGA Brightspace React course-files starter (Vite 6, React 19, `CourseContext`, zip packaging). [`frontend/`](frontend/) is **rebased from that template when needed**: copy scaffold files from the template folder, restore this project’s `package.json` name (`uga-elc-kaltura-transcript-importer`), then layer in the transcript-importer modules (`src/api`, `src/lib`, `ImporterApp`, caption-backend env vars). New Brightspace React tools can follow the same pattern—start from the template snapshot, then add an application-specific `src/` layer without forking the starter wholesale.

See [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) for n8n workflow setup, frontend build wiring, and course embedding.

Related notes: [`COURSE_TRANSCRIPTS_TO_FILES_HANDOFF.md`](COURSE_TRANSCRIPTS_TO_FILES_HANDOFF.md).
