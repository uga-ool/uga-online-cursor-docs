# Parquet Storage Strategy (Backend-First)

## Source Of Truth

Brightspace Course Files stores timeline artifacts:

- `timeline-data.parquet` (canonical)
- `timeline-manifest.json` (metadata and revision)
- `timeline-data.json` (editor hydration snapshot)
- `dist.zip` (deployable TimelineJS player package for course file embedding)

## Current Implementation Boundary

- App never writes parquet directly.
- App posts timeline DTO to backend.
- Backend validates and owns parquet serialization/upload.

## D2L Upload Flow

1. Initiate upload (`upload`) to receive `fileKey` and location.
2. Upload parquet bytes to returned upload URL.
3. Finalize save (`save`) to attach into course files path.

`server/src/storage/d2lCourseFilesApi.js` is the integration seam for this flow.

## Schema Versioning

- `timeline.metadata.schemaVersion` is required.
- `server/src/schema.js` applies migration hook when loaded schema lags current version.

## Scope And Attribution

- Artifact paths are scoped by `ou/moduleId`.
- Canonical timeline ID is `<ou>#<moduleId>`.
- Manifest stores `updatedByUserId`, `updatedBy`, `updatedAt`, and `revision` for submission traceability.
