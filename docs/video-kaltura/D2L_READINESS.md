# D2L (Brightspace) readiness — step-by-step

Use this after [VERIFY_LOCAL.md](VERIFY_LOCAL.md) basic setup. It mirrors the deployment plan: local gates, hosting, env, Lit embed, in-D2L smoke, governance.

## 1. Local verification (same build you ship)

With Djinn running (`npm run dev` or `npm start`):

```bash
npm run smoke
npm run verify:providers
```

- **`verify:providers`** — `GET /api/llm/providers` must return a non-empty **`configured`** array (e.g. `openai`) when using the embedded gateway.

End-to-end LLM + Kaltura (requires a **captioned** Kaltura `entryId` and valid `.env`):

```bash
export DJINN_TEST_ENTRY_ID='0_yourEntryId'
npm run verify:ask
```

That script sends **`Origin: https://elearning.uga.edu`** by default (production-like). If your **`ALLOWED_ORIGINS`** patterns differ, either align patterns or temporarily edit `scripts/verify-ask.sh` / pass a matching origin.

If Djinn has **`DJINN_RELAX_ORIGIN_DEV=true`**, you may use:

```bash
DJINN_RELAX_ORIGIN=1 DJINN_TEST_ENTRY_ID='0_...' npm run verify:ask
```

Do **not** use relax mode in production.

## 2. Hosting (HTTPS)

See [HOSTING.md](HOSTING.md) — **`npm run build`**, **`npm start`**, optional **Dockerfile**, reverse proxy, **`TRUST_PROXY`** behind a load balancer.

## 3. Production environment

Copy from [.env.example](../.env.example) into your host’s secret store. Checklist:

| Variable | Production |
|----------|------------|
| `KALTURA_PARTNER_ID`, `KALTURA_ADMIN_SECRET` | Production Kaltura |
| `LLM_OPENAI_API_KEY` (or other `LLM_*`) | Embedded gateway |
| `ALLOWED_ORIGINS` | D2L / Brightspace hostname patterns learners use |
| `DJINN_RELAX_ORIGIN_DEV` | **Unset / false** |
| `DJINN_API_KEY` | Optional; if set, Lit needs `djinn-api-key` |
| `REQUIRE_BRIGHTSPACE_SESSION` | Optional; if true, page must load inside Brightspace |
| `TRUST_PROXY` | `1` when behind one trusted reverse proxy |

Full narrative: [ELC_DEPLOY.md](ELC_DEPLOY.md).

## 4. Lit component (`uga-video`)

See [D2L_LIT_EMBED.md](D2L_LIT_EMBED.md).

## 5. Smoke test inside Brightspace

See [D2L_SMOKE_TEST.md](D2L_SMOKE_TEST.md).

## 6. Governance

See [GOVERNANCE.md](GOVERNANCE.md).

## Sign-off before D2L production

Use this as a final gate with your team (privacy / legal / IT as needed):

- [ ] **Secrets:** Production `KALTURA_*`, `LLM_*`, optional `DJINN_API_KEY` only in the deployed environment (not committed).
- [ ] **Origins:** `ALLOWED_ORIGINS` matches learner Brightspace hostnames; **`DJINN_RELAX_ORIGIN_DEV` is not `true`**.
- [ ] **Smoke:** [D2L_SMOKE_TEST.md](D2L_SMOKE_TEST.md) completed in a real course topic (player + Djinn ask + citations).
- [ ] **Governance:** [GOVERNANCE.md](GOVERNANCE.md) reviewed (logging, retention, FERPA / third-party LLM).

**Local automation (dev machine):** `npm run verify:local` (health + LLM providers), then `DJINN_TEST_ENTRY_ID=... npm run verify:ask` with a captioned entry.
