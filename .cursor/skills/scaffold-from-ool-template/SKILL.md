---
name: scaffold-from-ool-template
description: Scaffold a new Brightspace course-file React app under UGA-Brightspace-React-Apps/apps/ by copying from UGA-Brightspace-React-Template. Use when creating a new app, rebasing on the template, or the user mentions template snapshot or course-file starter.
---

# Scaffold from OOL React template

Copy patterns from the **read-only** template into **`UGA-Brightspace-React-Apps/apps/<app-name>/`**. Never scaffold by editing `UGA-Brightspace-React-Template` in place.

**Public template:** https://github.com/uga-ool/UGA-Brightspace-React-Template  
**Monorepo:** `UGA-Brightspace-React-Apps/apps/` (one npm package per folder)  
**Team docs:** `docs/course-file-react-apps/`, `docs/_meta/manifest.md`

## 1. Confirm target

- **Required path:** `UGA-Brightspace-React-Apps/apps/<app-name>/` (kebab-case folder name matching the app id)
- Do not write feature code under `UGA-Brightspace-React-Template`
- Do not scaffold new course-file apps as a new sibling repo
- **Exception:** Only use a legacy standalone repo (e.g. `uga-elc-kaltura-caption-import` with its own `frontend/`) when the user **explicitly** names that repo — not for greenfield scaffolds
- Follow existing apps’ `package.json`, `zip.js`, and build patterns in the monorepo

## 2. Read reference (read-only)

From `UGA-Brightspace-React-Template`, or an in-repo frozen folder such as `UGA-Brightspace-React-Template-main/` (see `docs/captions-and-transcripts/overview.md`), inspect and copy as needed:

- `vite.config.ts` (relative paths, no hashed filenames for Manage Files)
- `index.html`
- `src/context/CourseContext` (OU, XSRF, WhoAmI, role)
- `src/api/` helpers
- `.env.example` / env patterns
- Zip or build scripts from the template README

## 3. Copy scaffold

- Vite + TypeScript + SWC
- `dist/` layout suitable for Brightspace **Manage Files** (relative asset paths)
- Do not fork the entire template tree wholesale; copy only what the app needs

## 4. App layer

- Set `package.json` `name` to the app identifier
- Add app-specific `src/` (components, API routes, main view)
- Configure `VITE_API_BASE_URL` (typically `/d2l/api`), `VITE_LP_VERSION`, `VITE_LE_VERSION` per the app or monorepo README
- Use `import.meta.env.DEV` for local vs embedded eLC behavior

## 5. Design system

Load UGA Online Design System CDN in HTML shells per `elc-d2l-design-system` rule (`base.css`, `scripts.js`, documented Google Fonts).

## 6. Verify

- Run `npm run build` in the work app directory (`UGA-Brightspace-React-Apps/apps/<app-name>/`)
- Confirm `dist/` is ready for course-files upload (paths and zip layout per README)

## 7. Monorepo README (optional)

When the app is complete or the user asks, add a row to the `apps/` table in `UGA-Brightspace-React-Apps/README.md`.

## 8. PR note

Document intentional diffs from the public template in the PR description only when non-obvious.

## Policy

Respect `upstream-reference-repos.mdc`: template and canonical upstream repos stay read-only during this workflow.
