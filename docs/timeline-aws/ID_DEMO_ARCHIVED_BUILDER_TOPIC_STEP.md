# Archived content from `id-demo.html`

**Current product guidance:** deploy **`index.html` only** as a direct content topic from Manage Files (full page). **Iframe-based** embedding in a wrapper HTML page is **not** supported for this tool; the live ID demo (`app/public/id-demo.html`) documents the direct-file pattern only. The sections below preserve older text and iframe examples for historical reference.

## “Create a builder content topic” (former ID demo Step 2)

**Status:** Removed from `app/public/id-demo.html` (April 2026). The timeline builder does **not** require this workflow; many courses work by linking or opening `index.html` from Course Files without a separate “builder topic + iframe” step.

**Why it existed:** Optional instructional-design patterns for eLC: restrict who sees the authoring UI, control layout height, or wrap the tool in a parent HTML page.

**Why it was dropped:** Redundant with real-world usage (direct topic from uploaded files is sufficient). Keeping the copy here avoids losing embed examples or rationale.

---

## Former Step 2 copy (instructional text)

### Create a builder (ID-only) content topic

Create a topic that only instructional staff can access.

#### Recommended: iframe from Course Files

- Create a content topic (HTML page) and embed the builder in an iframe.
- Use the Manage Files path where you uploaded the package.

```html
<iframe
  src="/content/enforced4/YOUR_OU/TimelineJS/index.html"
  style="width:100%; height:900px; border:0;"
  title="Timeline Builder"
></iframe>
```

Replace `YOUR_OU` and the folder path to match your course files location.

#### Alternative: no iframe (topic from HTML file)

- Create the topic directly from `index.html` in Course Files.
- Edit `index.html` and set `timeline-readonly` as needed (see current ID demo “Read-only” step).

```html
<meta name="timeline-readonly" content="false" />
<!-- Use content="true" for viewer-only mode -->
```

---

## When the archived iframe pattern might be useful (not in the live ID demo)

- A program requires a **fixed-height** wrapper or extra chrome around the tool that a plain file topic cannot provide.
- You are prototyping **visibility separation** using a custom parent HTML shell (staff vs student) and accept iframe tradeoffs (sizing, cookies, CSP).

The maintained package assumes **direct `index.html`** topics instead. Treat iframe material in this file as historical reference only.

---

## Archived: id-demo.html lead — design system and Lit bundle notes

**Removed** from `app/public/id-demo.html` (April 2026). The page still loads the same assets; this copy was dropped from the visible intro only.

**Paragraph 1 (Design System + Lit components):** The instructional designer guide stated that the page used the same UGA Online Design System and UGA Lit Components (`<uga-toc>`, `<uga-code>`, `<uga-return-to-top>`) as other UGA Online demos, with a link to https://design.online.uga.edu/ .

**Paragraph 2 (Prism + bundle + build):** Code samples require Prism (loaded at the bottom of the page) plus `./js/uga-components.js` from the uga-lit-components bundle. Developers run `npm run build` in this app after building uga-lit-components so the bundle is copied into `app/public/js/` (see `app/scripts/copy-uga-components.mjs`). The builder `index.html` loads the same file from the app entry so `<uga-return-to-top>` upgrades after the module runs. In eLC the bundle may be hosted at `/shared/ugaonline/js/uga-components.js` instead; the script URL in `id-demo.html` or the loader in `app/src/main.tsx` would need updating for that path.

---

## Archived: read-only via iframe wrapper (removed from live ID demo)

The ID demo previously showed read-only mode using an **iframe** whose `src` pointed at `index.html?readOnly=true`. That pattern is no longer documented for this product; use **direct** `index.html` with a query string or `<meta name="timeline-readonly" ...>` instead.

```html
<iframe
  src="/content/enforced4/YOUR_OU/TimelineJS/index.html?readOnly=true"
  style="width:100%; height:750px; border:0;"
  title="Course Timeline"
></iframe>
```
