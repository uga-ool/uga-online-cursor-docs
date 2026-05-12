# UGA Timeline JS for D2L

This repository contains a D2L-focused TimelineJS builder for instructional designers.

## Important Boundary

- `TimelineJS3-master/` is vendored upstream source.
- Do not edit files inside `TimelineJS3-master/`.
- Any integration or customization must happen in `app/` and `server/`.

## Projects

- `app/`: Brightspace-hosted React UI (authoring, preview, publish).
- `server/`: Backend API for validation and Parquet + manifest persistence.

## Core runtime vs local development

- **Core production runtime (recommended)**: `template.yaml` deploys the backend as **Lambda + API Gateway + DynamoDB** (Option B). See [`docs/SAM_DEPLOYMENT.md`](docs/SAM_DEPLOYMENT.md).
- **Local development runtime (optional)**: `server/src/index.js` runs an Express server using `server/src/storage/localCourseFiles.js` to persist artifacts under `server/data/course-files/` for quick iteration without AWS.
  - This local server is **not** used by the SAM/Lambda deployment.
  - Do not treat local filesystem persistence as a production storage mechanism.

## Local Development

### App

```bash
cd app
npm install
npm run dev
```

### Server

```bash
cd server
npm install
npm run dev
```

## Deployment Notes

- Build app assets and upload `app/dist_zip/dist.zip` to Brightspace Course Files.
- In eLC / Brightspace **Content**, add topics that open **`index.html` from Course Files as the full page** (direct file topic). Do **not** embed the app inside a parent HTML `<iframe>`; that pattern is out of scope for this package. See `app/public/id-demo.html` after build for instructional steps.
- Deploy server to your approved backend environment.
- Configure app environment with backend base URL.
- Scope timelines by course/module using key format `<ou>#<moduleId>`.
- Use Brightspace `whoami.Identifier` as submitter ID when saving timeline revisions.
- Use the scoped API routes (`/timeline/{ou}/{moduleId}/...`) for all editor operations.
- For Option B (SAM + Lambda + API Gateway), use [`docs/SAM_DEPLOYMENT.md`](docs/SAM_DEPLOYMENT.md).
- Use [`docs/PRODUCTION_HARDENING_CHECKLIST.md`](docs/PRODUCTION_HARDENING_CHECKLIST.md) before production cutover.

## Option B Quick Commands

### Backend

```bash
cd server
npm install
npm run sam:build
npm run sam:deploy
```

### Frontend Production Package

```bash
cd app
cp .env.production.example .env.production
npm install
npm run build:prod
```

Upload `app/dist_zip/dist.zip` to Brightspace Course Files.

Optional ID-demo history: material removed from `app/public/id-demo.html` (former builder-topic iframe step, read-only iframe snippet, and the technical lead about Design System / Lit / Prism / bundle copy) is preserved in [`docs/ID_DEMO_ARCHIVED_BUILDER_TOPIC_STEP.md`](docs/ID_DEMO_ARCHIVED_BUILDER_TOPIC_STEP.md).
