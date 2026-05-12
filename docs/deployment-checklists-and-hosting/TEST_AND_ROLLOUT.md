# Test And Rollout Plan

## Unit Tests (next step)

- `app/src/utils/timelineAdapter.ts` DTO -> TimelineJS payload mapping
- `server/src/schema.js` validation and migration behavior

## Integration Tests (next step)

- Save/load lifecycle against scoped backend API (`/timeline/{ou}/{moduleId}/...`)
- D2L upload adapter tests with mocked upload/save endpoints
- Public TimelineJS payload response from `GET /timeline/{ou}#{moduleId}`
- Lambda whoami verification rejects mismatched `X-User-Id` with `403`
- Lambda whoami verification accepts matching Brightspace user identity

## UAT Checklist

- Author timeline from scratch
- Edit existing timeline
- Publish and reopen from course files
- Validate role restrictions (student read-only)
- Confirm timeline preview renders media and dates
- Confirm saves in module A do not overwrite module B in same course
- Confirm manifest stores submitter user ID and display name
- Confirm API responses include strict CORS allow-origin matching deployment domain
- Confirm CloudWatch logs are retained per configured retention days

## Rollout

1. Pilot in limited courses with instructional designers.
2. Monitor save/load failures and schema mismatch logs.
3. Lock CORS origin allow-list to Brightspace deployment domain.
4. Enforce least-privilege IAM on Lambda/API Gateway/DynamoDB resources.
5. Align Lambda + DynamoDB regions before production cutover.
6. Enable broader course rollout after acceptance criteria pass.

## Option B Operational Checks

- Verify SAM stack output `TimelineApiUrl` is wired to `VITE_TIMELINE_BACKEND_BASE`.
- Verify `GET /timeline/{timelineId}` can be consumed by TimelineJS embed page.
- Verify Lambda write handler can reach Brightspace whoami endpoint in deployed network path.
