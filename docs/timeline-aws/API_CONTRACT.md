# Timeline API Contract

Base URL: `VITE_TIMELINE_BACKEND_BASE`

## Scoped editor routes (preferred)

These routes scope timeline data by **course (`ou`) + module (`moduleId`)**.

## `GET /timeline/{ou}/{moduleId}/manifest`

Returns the latest timeline manifest.

## `GET /timeline/{ou}/{moduleId}/load`

Returns the latest timeline draft JSON payload for editor hydration.

## `POST /timeline/{ou}/{moduleId}/validate`

Request body:

```json
{
  "timeline": {
    "metadata": {
      "title": "Course Timeline",
      "subtitle": "Optional",
      "schemaVersion": "1.0.0"
    },
    "events": []
  }
}
```

Response body:

```json
{
  "valid": true,
  "errors": []
}
```

## `POST /timeline/{ou}/{moduleId}/save`

Validates then saves timeline artifacts and returns manifest metadata.

Required identity/context:

- Header: `X-User-Id` (`whoami.Identifier`)
- Optional header: `X-User-Display-Name`
- Optional request body context:
  - `context.userId`
  - `context.topicId`

Response includes:

- `manifest`
- `d2lUpload` (local dev stub now; integrate real upload/save flow in deployment env)

## Public TimelineJS read route

## `GET /timeline/{timelineId}`

`timelineId` format: `<ou>#<moduleId>`

Returns TimelineJS-compatible JSON:

```json
{
  "title": {
    "text": {
      "headline": "Course Timeline",
      "text": "Optional subtitle"
    }
  },
  "events": []
}
```

## Legacy compatibility routes

The following still exist for older clients and default to `moduleId = course-root`:

- `GET /timeline/{ou}/manifest`
- `GET /timeline/{ou}/load`
- `POST /timeline/{ou}/validate`
- `POST /timeline/{ou}/save`
