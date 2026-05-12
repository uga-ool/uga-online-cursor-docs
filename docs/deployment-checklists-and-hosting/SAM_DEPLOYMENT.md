# SAM Deployment Runbook (Option B)

This runbook deploys the Timeline backend as Lambda + API Gateway + DynamoDB using `template.yaml`.

## 1) Prerequisites

- AWS CLI configured with deployment credentials
- AWS SAM CLI installed
- Node.js available for Lambda packaging

## 2) Required configuration values

Collect and confirm before deployment:

- `EnvironmentName` (example: `dev`, `stage`, `prod`)
- `BrightspaceBaseUrl` (example: `https://uga.view.usg.edu`)
- `LPVersion` (default `1.47`)
- `CorsAllowOrigin` (exact Brightspace origin, no wildcard in prod)
- `EventsTableName` / `MetaTableName` (defaults are fine if unique in account)
- `LogRetentionDays` (recommended 14+)

## 3) Build and deploy backend (SAM)

From `server/`:

```bash
npm install
npm run sam:build
npm run sam:deploy
```

For guided deploy, enter parameter values when prompted.

## 4) Verify deployed API routes

After deploy, use the stack output `TimelineApiUrl`:

- `GET {TimelineApiUrl}/timeline/{ou}/{moduleId}/manifest`
- `GET {TimelineApiUrl}/timeline/{ou}/{moduleId}/load`
- `POST {TimelineApiUrl}/timeline/{ou}/{moduleId}/validate`
- `POST {TimelineApiUrl}/timeline/{ou}/{moduleId}/save`
- `GET {TimelineApiUrl}/timeline/{timelineId}`

## 5) Frontend production wiring

Create `app/.env.production` from `app/.env.production.example` and set:

- `VITE_TIMELINE_BACKEND_BASE={TimelineApiUrl}`

Then build deployable course package:

```bash
cd app
npm install
npm run build:prod
```

Upload `app/dist_zip/dist.zip` into Brightspace Course Files.

## 6) Write authentication behavior (whoami verification)

`POST /save` enforces Brightspace whoami verification in Lambda:

- Lambda forwards incoming `Authorization` or `Cookie` header to Brightspace whoami
- verified `Identifier` must match incoming `X-User-Id` / `context.userId`
- mismatch is rejected with `403`

## 7) UAT checklist

- Save in module A does not overwrite module B in the same course
- Save in course X does not overwrite course Y with same module id
- Manifest returns `updatedByUserId`, `updatedBy`, `revision`, `updatedAt`
- `GET /timeline/{ou}#{moduleId}` returns valid TimelineJS JSON
- CORS only allows approved Brightspace origin(s)

## 8) Production hardening

- Keep Lambda and DynamoDB in the same AWS region
- Use least-privilege IAM policies only for needed DynamoDB actions
- Set CloudWatch retention in template parameter and monitor errors
