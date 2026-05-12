# Smoke test inside D2L (Brightspace)

Run this **after** Djinn is on **HTTPS**, **`ALLOWED_ORIGINS`** matches production, and **`uga-video`** is deployed with **`djinn-api-base`** pointing at that host.

## Prerequisites

- [ ] Djinn **`GET /health`** returns `ok` on the production URL.
- [ ] **`DJINN_RELAX_ORIGIN_DEV`** is **not** enabled in production.
- [ ] Course topic loads **`uga-components.js`** and includes **`uga-video`** with **`enable-djinn`** and correct **`djinn-api-base`**.

## Steps

1. **Open the content topic** as a learner (or preview) on the **same hostname** you configured in **`ALLOWED_ORIGINS`**.
2. **Confirm the Kaltura player** loads for the entry.
3. **Open the Kaltura Djinn** panel (below the player when enabled).
4. **Ask a short question** about content that appears in captions (e.g. “What does the speaker say about …?”).
5. **Confirm** you get an **answer** and, if citations appear, use **Jump to** / seek controls and verify the timestamp lands in the right segment.

## Failure triage

| Symptom | Likely cause | Action |
|--------|----------------|--------|
| **403** on ask | Origin guard | Add/fix **`ALLOWED_ORIGINS`** pattern for the page’s **`Origin`** host (include `www` vs non-`www` if both exist). |
| **401** on ask | **`DJINN_API_KEY`** mismatch | Align Lit **`djinn-api-key`** with server env or remove key on both sides for testing only. |
| **502** on ask | Kaltura or LLM | Check Djinn server logs: caption fetch vs **`POST /api/llm/chat`** upstream. Verify Kaltura secret and LLM key in **production** env. |
| **400** / empty answer | No captions | Pick an entry with human captions; machine audio-only may not expose WebVTT the same way. |
| Works in relax dev, fails in prod | **`DJINN_RELAX_ORIGIN_DEV`** | Expected; fix origins instead of re-enabling relax in production. |

## Optional strict session test

If **`REQUIRE_BRIGHTSPACE_SESSION=true`**:

- Repeat the ask **from inside Brightspace** (not raw `curl` without token).
- Confirm **`uga-video`** is on an authenticated eLC page so **`X-Csrf-Token`** is sent.
