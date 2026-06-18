# eLC test plan generator

Copy into Cursor **Ask** mode when QA needs a test plan for a feature or PR.

---

Generate an eLC (Brightspace) test plan checklist for the following feature:

**Feature name:**

**Description:**

**Tool type:** <!-- course-file React app / LTI widget / agent plugin / Lit component / other -->

**Manage Files or launch path:**

**Roles to test:** <!-- instructor, student, admin -->

Use sections from `docs/deployment-checklists-and-hosting/D2L_SMOKE_TEST_CHECKLIST.md`:

1. **Functional** — load, required fields, generation/output, copy/download
2. **Accessibility** — keyboard-only workflow, focus states, status announcements
3. **Integration** — design system assets, API/proxy calls, console errors
4. **Regression and rollback** — compare to legacy behavior if applicable; rollback URL

Output markdown checkboxes suitable for a GitHub PR **Test plan** section.

Use test-sandbox OU placeholders only — no real student data.
