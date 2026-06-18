# HTML remediation review

Copy into Cursor **Ask** or **Agent** mode with the HTML file @-referenced.

---

Review `@<path-to-file.html>` for accessibility, semantic HTML, and **UGA Online Design System** compliance suitable for eLC (Brightspace) course content.

Apply standards from:
- `.cursor/rules/uga-online-html-cleanup.mdc`
- `.cursor/rules/uga-online-design-system.mdc`
- `docs/pdf-accessibility/`

Check:
- **Design system (required):** Google Fonts, `base.css`, skip link, `<main id="main-content">`, `obj-reading-width` (or other documented utilities), `scripts.js` on full pages; design-system wrappers/classes on fragments
- Image alt text
- Heading hierarchy (no skipped levels; one h1)
- Table headers and scope
- Link text quality
- lang attribute on html
- Equations (MathML or review flags)
- PDF/PPT artifacts (page breaks, presenter notes, bullet paragraphs)
- No legacy inline font/color styles that bypass the design system

Report findings as **blocking**, **should fix**, or **suggestion**. Do not remove instructional content.

If I ask you to fix issues, return corrected HTML that **always** uses the UGA Online Design System, with a `<!-- REVIEW SUMMARY: ... -->` at the bottom for any items needing human review.

No student PII in examples or output.
