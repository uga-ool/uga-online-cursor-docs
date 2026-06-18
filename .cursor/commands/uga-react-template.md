---
description: Scaffold or extend a course-file React app (UGA-Brightspace-React-Template)
---

You are helping with a **Brightspace course-file** React app based on the UGA Online template.

**Upstream reference:** https://github.com/uga-ool/UGA-Brightspace-React-Template

1. Match **Vite + TypeScript** patterns: `CourseContext`, API helpers, and `import.meta.env.DEV` for local vs eLC.
2. Configure **`VITE_API_BASE_URL`**, **`VITE_LP_VERSION`**, and **`VITE_LE_VERSION`** per the app README; use `/d2l/api` when embedded in eLC unless documented otherwise.
3. **Build:** `npm run build` should emit a **`dist/`** tree suitable to upload to **Manage Files** (preserve relative paths; do not nest `dist/` incorrectly inside zips if the README specifies a flat layout).
4. When adding UI, prefer the **UGA Online Design System** CDN (`https://design.online.uga.edu/css/base.css` and `js/scripts.js`) plus the documented Google Fonts — see https://design.online.uga.edu/getting-started/installation/
5. If this monorepo’s **`templates/`** or **`apps/`** copy differs from the public template, follow the **local README** and call out intentional differences in comments or PR text only when necessary.

Execute the user’s task; keep changes scoped to the files they care about.
