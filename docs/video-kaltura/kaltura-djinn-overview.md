# Kaltura Djinn

Caption-grounded Q&A for Kaltura videos embedded in **eLC (Brightspace)**. Learner questions are answered using retrieved caption excerpts. **LLM calls use the same `POST /api/llm/chat` contract as the Brightspace agent framework**, but this repo **embeds that gateway** by default—vendor API keys stay on the server, never in the browser.

## Embedded LLM gateway (default)

This service mounts:

- **`POST /api/llm/chat`** — forwards to OpenAI / OpenRouter / Airia / UGA gateway using **`LLM_OPENAI_API_KEY`** (or the matching `LLM_*_API_KEY` for your provider) in `.env`.
- **`GET /api/llm/providers`** — lists which providers have keys configured.

If **`LLM_GATEWAY_BASE_URL` is unset**, `/v1/ask` calls the gateway on **the same port** (`http://127.0.0.1:<PORT>`). You only need **one** `npm run dev` process.

Optional: set **`LLM_GATEWAY_BASE_URL`** to a **separate** framework backend URL if you want keys and LLM traffic on another host (same as `VITE_BACKEND_URL` in agent proxy mode).

## Not AGENT-GUIDE

**AGENT-GUIDE.md** in the [agent framework repo](https://github.com/uga-ool/uga-online-brightspace-agent-framework) is for **React agent plugins** (`frontend/src/agents/...`). **Kaltura Djinn does not add a new agent**—it is a standalone Node API plus optional shared gateway routing.

## Features

- `POST /v1/ask` — `{ "entryId", "question" }` → `{ "answer", "citations" }` with millisecond timestamps for seek/jump.
- Kaltura: admin session → `caption_captionasset` list + serve → WebVTT/SRT parse.
- RAG v0: chunk captions, keyword overlap retrieval, JSON-structured model output.
- eLC guard: `Origin` / `Referer` hostname must match `ALLOWED_ORIGINS` patterns (unless `DJINN_RELAX_ORIGIN_DEV=true`).
- Optional `DJINN_API_KEY` for an extra shared-secret gate.
- Rate limiting per client IP on `/v1/ask` and on `/api/llm/*`.

## Quick start

```bash
cp .env.example .env
# edit .env — Kaltura admin creds, LLM_OPENAI_API_KEY (embedded gateway), ALLOWED_ORIGINS

npm install
npm run dev
```

Health: `GET http://localhost:3847/health` (includes `llmGateway`: `embedded` | `external`).

Step-by-step local checks: [docs/VERIFY_LOCAL.md](docs/VERIFY_LOCAL.md). Quick chain: **`npm run verify:local`**, then **`DJINN_TEST_ENTRY_ID=... npm run verify:ask`** (captioned entry).

**Deploy to D2L / Brightspace:** [docs/D2L_READINESS.md](docs/D2L_READINESS.md) (hosting, env, Lit embed, smoke test, governance).

## eLC smoke test (checklist)

After Djinn is deployed:

1. **CORS / origins:** Djinn `ALLOWED_ORIGINS` includes the Brightspace hostname learners use; production must **not** use `DJINN_RELAX_ORIGIN_DEV`.
2. **Lit embed:** Use `<uga-video … enable-djinn djinn-api-base="https://<your-djinn-host>">` (and `djinn-api-key` if enabled).
3. **Xsrf:** If **`REQUIRE_BRIGHTSPACE_SESSION=true`**, confirm questions work in eLC ( **`uga-video`** forwards `X-Csrf-Token` to Djinn and on to `/api/llm/chat`).
4. **Captions:** Pick a Kaltura entry that already has captions.

## LLM gateway auth

`validateSession` on `/api/llm/chat` allows requests without `X-Csrf-Token` unless **`REQUIRE_BRIGHTSPACE_SESSION=true`**. The Lit **`uga-video`** client forwards `X-Csrf-Token` from Brightspace when available.

## Related repos

- **uga-lit-components** — `<uga-video enable-djinn djinn-api-base="https://djinn...">` calls this API.
- **[uga-online-brightspace-agent-framework](https://github.com/uga-ool/uga-online-brightspace-agent-framework)** — optional standalone backend if you set **`LLM_GATEWAY_BASE_URL`** instead of using the embedded gateway.

Gateway implementation in this repo lives under **`src/llm-gateway/`** (ported from the framework backend proxy stack).

See [docs/API.md](docs/API.md) for the HTTP contract.

## License

Internal UGA use unless otherwise noted.
