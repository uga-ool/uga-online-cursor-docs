---
description: Apply UGA Online Design System in eLC (D2L) HTML or Vite index
---

You are working on a University of Georgia **eLC** (D2L Brightspace) surface (course files, custom HTML, or a Vite app shell).

1. Follow **UGA Online Design System** installation: https://design.online.uga.edu/getting-started/installation/
2. Load **CSS** from `https://design.online.uga.edu/css/base.css` (or a **versioned** URL like `https://design.online.uga.edu/v1.x.x/css/base.css` for production stability).
3. Load **JS** from `https://design.online.uga.edu/js/scripts.js` (or matching versioned path) before `</body>`.
4. Add the **Google Fonts** links for Merriweather, Merriweather Sans, and Oswald exactly as documented on that page.
5. For full pages, include **skip to main content** (`cmp-skip-to-content`), `<main id="main-content">`, and layout classes such as **`obj-reading-width`** where appropriate (see the minimal template on the installation page).
6. Preserve **Brightspace** constraints: relative asset paths for course-file builds, no breaking changes to `VITE_*` API base or LP/LE versions unless the task explicitly requires it.

Implement or adjust the files the user indicated; keep the diff minimal and match existing project patterns.
