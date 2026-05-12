# D2L / eLC — `uga-video` + Kaltura Djinn

Learners call Djinn from the browser via **`uga-video`** in **[uga-lit-components](https://github.com/uga-ool/uga-lit-components)** (or your fork).

## 1. Build and publish the Lit bundle

In the **uga-lit-components** repo:

```bash
npm install
npm run build
```

Upload the built **`uga-components.js`** (path per that repo’s `README` / setup guide) to **eLC Public Files** or your CDN, and load it from the course topic with a module script, same as other Lit components.

## 2. Markup in course content

Use your real Djinn **HTTPS** origin (no trailing slash on `djinn-api-base`).

```html
<uga-video
  videoid="YOUR_KALTURA_ENTRY_ID"
  enable-djinn
  djinn-api-base="https://djinn.yourorg.edu"
></uga-video>
```

If Djinn has **`DJINN_API_KEY`** set, add the attribute (treat the value as sensitive):

```html
  djinn-api-key="your-shared-secret"
```

Component attributes (see `src/components/uga-video.ts`):

| Attribute | Purpose |
|-----------|---------|
| `videoid` | Kaltura entry id |
| `enable-djinn` | Enables Djinn panel |
| `djinn-api-base` | Djinn service origin |
| `djinn-api-key` | Optional `X-Djinn-Api-Key` header |

The component **`POST`s** to **`{djinn-api-base}/v1/ask`** and forwards **`X-Csrf-Token`** when present (Brightspace), which matters if **`REQUIRE_BRIGHTSPACE_SESSION=true`** on Djinn.

## 3. ALLOWED_ORIGINS

Djinn must allow the **Origin** host of the Brightspace page (see [ELC_DEPLOY.md](ELC_DEPLOY.md)). If you get **403** on ask, the pattern list does not match the browser’s **`Origin`** hostname.

## 4. Demo

If the Lit repo is checked out next to Djinn, `npm run preflight:elc` checks that `demo/video.html` still references **`enable-djinn`** for manual UI testing.
