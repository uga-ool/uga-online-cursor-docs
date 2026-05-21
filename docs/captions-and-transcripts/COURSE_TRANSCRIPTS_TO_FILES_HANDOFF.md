# Handoff: Course Kaltura transcripts → Brightspace course Files (no AI)

Use this document to start a **new Cursor chat / plan** for a **smaller, separate project**. Goal is **batch export**: gather **caption transcripts** for **all Kaltura videos used in a course** and **upload or publish them into that course’s Files** (or equivalent managed content area in Brightspace / eLC).

**Explicit non-goals**

- No LLM, no Q&A, no RAG, no learner-facing chat.
- No learner-facing caption Q&A; this is a **different product** (script, small service, or Brightspace integration).

---

## Problem statement

Instructors or admins want **accessible, searchable text** of what was said in embedded Kaltura videos **in one place**—the course **Files** folder—without manually downloading captions per video.

---

## Technical anchors (existing code to reuse as reference)

Repo **uga-elc-kaltura-caption-import** (or a new worker) should implement **server-side** Kaltura admin session + caption retrieval. Use Kaltura API docs for:

| Concern | API |
|--------|-----|
| List caption assets for an entry | `caption_captionasset` `list` |
| Download caption file (WebVTT/SRT/etc.) | `caption_captionasset` `serve` |
| Env pattern | `KALTURA_PARTNER_ID`, `KALTURA_ADMIN_SECRET`, optional `KALTURA_SERVICE_URL` |

---

## Likely building blocks (to decide in planning)

1. **Enumerate “videos in this course”**
   - Parse HTML/topics for embedded players / known patterns (`entryId`, `widgetId`, etc.).
   - Or use **Brightspace APIs** (Content, HTML topics, LTI links) if available in your environment.
   - Or maintain a **CSV/list of entry IDs** per course (simplest MVP).

2. **Resolve transcripts**
   - For each Kaltura `entryId`: list caption assets; pick language/locale policy (first available, `en`, etc.); `serve` raw caption text.
   - Normalize output: **plain text**, **WebVTT**, or **SRT**—match what Files + accessibility workflow expect.

3. **Write to course Files**
   - **Brightspace Files API** (or equivalent REST) with OAuth/service user—**confirm exact endpoints and permissions** for your tenant.
   - Naming: e.g. `transcripts/<sanitized-title>-<entryId>.vtt`.
   - Idempotency: skip if file already exists or checksum matches.

4. **Runtime / auth**
   - **Batch script** run by admin (Node/Python) with Kaltura admin secret + Brightspace token.
   - Or **scheduled job** / AWS Lambda with secrets manager.
   - Avoid storing admin secrets in course content; keep server-side only.

---

## Suggested MVP slice

1. Input: **course identifier** + **list of Kaltura entry IDs** (manual CSV OK).
2. For each ID: fetch captions → save **one file per video** to local disk (prove pipeline).
3. Phase 2: upload those files to **Brightspace course Files** via API (after spike).

---

## Open questions for the planning chat

1. **Source of truth for “videos in course”** — automated discovery vs curated list?
2. **Brightspace version / API** — which APIs are approved for file upload in your org?
3. **Caption policy** — multiple tracks per entry (language); default language?
4. **Ownership** — who runs the job (instructor, TA, central admin)?
5. **FERPA / retention** — transcripts in Files may duplicate sensitive material; align with policy.

---

## Related repos (UGA local paths — adjust if cloned elsewhere)

- **Kaltura captions:** `uga-elc-kaltura-caption-import`.
- **Brightspace agents / LMS context:** `uga-online-brightspace-agent-framework` (may have OAuth/tool patterns; **not required** if using standalone REST).
- **Lit embeds:** `uga-lit-components` (`uga-video`) — useful for **HTML patterns** of how videos appear in content, not for transcript pipeline logic.

---

## Success criteria (draft)

- Given a course + entry ID list, produce **complete transcript files** for every entry that has captions, with clear errors for missing captions.
- Files appear in the **designated course folder** with predictable names.
- **No AI services** in the dependency chain.

---

*Generated as a handoff starter for a separate “transcripts only” project.*
