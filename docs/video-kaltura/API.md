# Kaltura Djinn HTTP API

Base URL: `https://<djinn-host>` (Djinn API is under `/v1`). Optional `LLM_GATEWAY_BASE_URL` must have no trailing slash when set.

## CORS

Browser calls from eLC must use an allowed origin (see `ALLOWED_ORIGINS`). Preflight sends `Content-Type`, `Authorization`, `X-Djinn-Api-Key`, `X-Csrf-Token`, `X-Xsrf-Token`.

## `GET /health`

Returns JSON:

```json
{
  "status": "ok",
  "service": "Kaltura Djinn",
  "version": "0.1.0",
  "llmGateway": "embedded"
}
```

`llmGateway` is `embedded` when `/v1/ask` uses the in-process `POST /api/llm/chat` on the same server, or `external` when `LLM_GATEWAY_BASE_URL` points elsewhere.

No origin check (for load balancers).

## `POST /v1/ask`

**Headers**

- `Content-Type: application/json`
- `X-Csrf-Token` (recommended): Brightspace referrer token; forwarded to the LLM Gateway when calling `/api/llm/chat`.
- `X-Djinn-Api-Key` or `Authorization: Bearer <key>`: required if `DJINN_API_KEY` is set on the server.

**Body**

```json
{
  "entryId": "0_abc123",
  "question": "What is the main idea of the introduction?"
}
```

**Success `200`**

```json
{
  "answer": "…",
  "citations": [
    { "startMs": 12000, "endMs": 18500, "text": "…" }
  ]
}
```

**Errors**

| Status | Meaning                                      |
| ------ | -------------------------------------------- |
| 400    | Missing fields, no captions, unparseable VTT |
| 401    | Invalid Djinn API key                        |
| 403    | Origin / Referer not allowed                 |
| 429    | Rate limited                                 |
| 502    | Kaltura or LLM gateway failure               |
| 503    | `ALLOWED_ORIGINS` empty in strict mode       |

```json
{ "error": "message" }
```

## LLM gateway integration

Djinn builds an OpenAI-style `messages` array and `POST`s to:

`{LLM_GATEWAY_BASE_URL}/api/llm/chat`

By default **`LLM_GATEWAY_BASE_URL` is omitted** and resolves to this same service (`http://127.0.0.1:<PORT>`), which exposes **`POST /api/llm/chat`** with the same contract as the Brightspace agent framework backend. Set **`LLM_OPENAI_API_KEY`** (or another provider key) in `.env` for that embedded gateway.

To use a **separate** gateway instead, set **`LLM_GATEWAY_BASE_URL`** to that API origin (no trailing slash). Keys then live on that service, not in Djinn’s `.env`.

Request shape: `messages`, `provider`, optional `model`. Keys stay server-side, never in the browser.
