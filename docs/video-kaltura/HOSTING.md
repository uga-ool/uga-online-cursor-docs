# Hosting Kaltura Djinn (HTTPS)

Djinn is a stateless **Node** API. Production traffic should use **HTTPS**; terminate TLS at your load balancer, reverse proxy, or platform ingress.

## Build and run

```bash
npm ci
npm run build
npm start
```

Default listen port comes from **`PORT`** (see `.env.example`; default **3847** in code). Set **`PORT`** in the environment if your platform assigns a dynamic port.

Health check for orchestration: **`GET /health`** (expect `status: ok`).

## Docker

From the repo root (requires a running Docker daemon):

```bash
npm run docker:build
# or: docker build -t kaltura-djinn .
docker run --rm -p 3847:3847 --env-file .env kaltura-djinn:local
```

Pass **production** secrets via your platform’s secret store or `--env-file`, not committed files.

## Reverse proxy

Point your public hostname to the Node process (or container). Example **nginx** location (TLS at `server` level):

```nginx
location / {
    proxy_pass http://127.0.0.1:3847;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

Djinn reads **`req.ip`** for rate limiting. Behind a reverse proxy, set **`TRUST_PROXY=1`** (or `true`) in the environment so Express honors **`X-Forwarded-For`** for client IP (single trusted hop). Adjust if your platform uses a different forwarding model.

## Production environment

See [ELC_DEPLOY.md](ELC_DEPLOY.md) and [D2L_READINESS.md](D2L_READINESS.md). Required categories:

- **Kaltura:** `KALTURA_PARTNER_ID`, `KALTURA_ADMIN_SECRET`
- **LLM (embedded):** e.g. `LLM_OPENAI_API_KEY`
- **Browser access:** `ALLOWED_ORIGINS` matching Brightspace; **do not** set `DJINN_RELAX_ORIGIN_DEV` in production
