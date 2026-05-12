# Production Hardening Checklist (Option B)

Use this checklist before promoting from pilot to wider release.

## Security

- [ ] `CorsAllowOrigin` is set to exact Brightspace domain(s), not `*`
- [ ] `GET /timeline/{timelineId}` visibility decision documented (public vs restricted)
- [ ] Lambda IAM scopes limited to required DynamoDB table actions
- [ ] Brightspace whoami verification enabled for save endpoints

## Reliability

- [ ] Lambda and DynamoDB are in the same region
- [ ] CloudWatch log retention configured (recommended 14+ days)
- [ ] Alarm strategy defined for elevated 4xx/5xx rates
- [ ] Save route tested with at least 2 concurrent editors

## Data Integrity

- [ ] `timeline_id` key uses `<ou>#<moduleId>` for all writes
- [ ] Save in one module does not alter timeline in another module
- [ ] Save in one course does not alter timeline in another course
- [ ] Manifest fields verified: `updatedByUserId`, `updatedBy`, `updatedAt`, `revision`

## Deployment Process

- [ ] `sam build` and `sam deploy --guided` succeed in target account
- [ ] Stack output `TimelineApiUrl` recorded in deployment notes
- [ ] `app/.env.production` points `VITE_TIMELINE_BACKEND_BASE` to deployed API stage
- [ ] `app/dist_zip/dist.zip` uploaded and validated in Brightspace course files
