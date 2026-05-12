# Kaltura Djinn — eLC (Brightspace) deployment checklist

**End-to-end order:** [D2L_READINESS.md](D2L_READINESS.md) (local checks → hosting → D2L smoke → governance).

Use this after local smoke tests pass ([VERIFY_LOCAL.md](VERIFY_LOCAL.md)).

## 1. Host Djinn

- Run the Node service on a URL reachable from learner browsers (HTTPS in production).
- Set **`ALLOWED_ORIGINS`** to Brightspace hostname patterns (e.g. `*.yourdomain.edu`, `yourdomain.edu`). Do **not** use `DJINN_RELAX_ORIGIN_DEV` in production.
- Set **`DJINN_API_KEY`** if you want a shared secret; then configure the Lit component with **`djinn-api-key`**.

## 2. LLM gateway

- **Default:** Djinn serves **`POST /api/llm/chat`** on the same host. Set **`LLM_OPENAI_API_KEY`** (or the matching provider key) in Djinn’s environment; you do **not** need a separate framework backend unless you choose to split them.
- **Optional split:** Deploy the [Brightspace agent framework](https://github.com/uga-ool/uga-online-brightspace-agent-framework) **backend** (or another compatible URL), then set Djinn **`LLM_GATEWAY_BASE_URL`** to that API’s **origin** (no trailing slash), same as agent **`VITE_BACKEND_URL`** in proxy mode. Keys then live on that service.
- Djinn calls the gateway **server-to-server**; browser CORS to the gateway is not required for `/v1/ask`.

## 3. Session strictness

- If **`REQUIRE_BRIGHTSPACE_SESSION=true`** on Djinn (embedded gateway), learners must hit Djinn from eLC so **`uga-video`** can attach **`X-Csrf-Token`**; Djinn forwards it to **`POST /api/llm/chat`**.

## 4. Lit / uga-video

- Build and publish **`uga-lit-components`** (`npm run build` in that repo).
- In course content, use for example:

```html
<uga-video
  videoid="YOUR_ENTRY_ID"
  enable-djinn
  djinn-api-base="https://djinn.yourorg.edu"
></uga-video>
```

- If **`DJINN_API_KEY`** is set, add **`djinn-api-key="..."`** (treat as sensitive; prefer server-injected or managed config).

## 5. Verify in Brightspace

1. Open a content topic with the embed; confirm the Kaltura player loads.
2. Open **Kaltura Djinn**, ask a question; confirm an answer and **Jump to** seek.
3. If requests fail with **403**, fix **`ALLOWED_ORIGINS`** to match the page’s **`Origin`** host.
4. If **502** on ask, check Djinn logs (Kaltura or LLM gateway).

## 6. Governance

- Use [GOVERNANCE.md](GOVERNANCE.md) as a starting checklist (logging, retention, FERPA, third-party LLM). Complete with your privacy/legal office.
