# Local verification (Kaltura Djinn)

Djinn is **not** an agent plugin from **AGENT-GUIDE.md**. By default this repo **embeds** the LLM gateway (`POST /api/llm/chat`) on the **same** process as `/v1/ask`, so you usually run **one** server.

## OpenAI (embedded gateway — default)

| Where | Set |
|--------|-----|
| **Djinn `.env`** | **`LLM_OPENAI_API_KEY`**, **`KALTURA_*`**, **`ALLOWED_ORIGINS`** (or **`DJINN_RELAX_ORIGIN_DEV=true`** for loose local curl) |
| Optional | **`LLM_GATEWAY_PROVIDER`**, **`LLM_GATEWAY_MODEL`**, **`LLM_MODEL`** / **`LLM_PROVIDER`** for the embedded upstream |

Do **not** set **`LLM_GATEWAY_BASE_URL`** unless you want `/v1/ask` to call a **different** gateway host.

For plain `curl` to Djinn locally, leave **`REQUIRE_BRIGHTSPACE_SESSION`** unset or `false`, or pass `X-Csrf-Token` (see `docs/API.md`).

Automated smoke (after the server is up): `DJINN_TEST_ENTRY_ID=<captioned_entry> npm run verify:ask` from the Djinn repo (optional `DJINN_RELAX_ORIGIN=1` only if Djinn has `DJINN_RELAX_ORIGIN_DEV=true`).

## 1. Kaltura Djinn (single process)

```bash
cd uga-kaltura-djinn
cp .env.example .env
# Set LLM_OPENAI_API_KEY, KALTURA_PARTNER_ID, KALTURA_ADMIN_SECRET
# For curl without a browser Origin, add DJINN_RELAX_ORIGIN_DEV=true
npm install
npm run dev
```

Confirm:

- `GET http://localhost:3847/health` — expect `"llmGateway": "embedded"` when `LLM_GATEWAY_BASE_URL` is unset.
- `GET http://localhost:3847/api/llm/providers` — expect `configured` to include `openai` when the key is set.
- Or run: `npm run verify:providers` (same check; optional `DJINN_BASE=...`).

## 2. Optional: external framework backend instead

If **`LLM_GATEWAY_BASE_URL=http://localhost:3001`** (or another origin), run the [framework `backend`](https://github.com/uga-ool/uga-online-brightspace-agent-framework) separately and put **`LLM_OPENAI_API_KEY`** there; Djinn’s `.env` then does **not** need that key for `/v1/ask`.

## 3. POST /v1/ask

With `DJINN_RELAX_ORIGIN_DEV=true`:

```bash
curl -sS -X POST http://localhost:3847/v1/ask \
  -H 'Content-Type: application/json' \
  -d '{"entryId":"<KALTURA_ENTRY_WITH_CAPTIONS>","question":"What is this video about?"}'
```

With strict origins, send a browser-like `Origin`:

```bash
curl -sS -X POST http://localhost:3847/v1/ask \
  -H 'Content-Type: application/json' \
  -H 'Origin: https://elearning.uga.edu' \
  -d '{"entryId":"<ENTRY_ID>","question":"Summarize the introduction."}'
```

If `DJINN_API_KEY` is set, add `-H "X-Djinn-Api-Key: <key>"`.

Expect `200` with `answer` and `citations`. `400` often means no captions; `502` often means Kaltura or LLM failure — check the Djinn terminal logs.

## 4. Session strictness

If **`REQUIRE_BRIGHTSPACE_SESSION=true`**, Djinn must receive **`X-Csrf-Token`** on `/v1/ask` and forwards it to **`POST /api/llm/chat`**. The **`uga-video`** Lit component does that when embedded in Brightspace. Plain `curl` without a token may fail in that mode.
