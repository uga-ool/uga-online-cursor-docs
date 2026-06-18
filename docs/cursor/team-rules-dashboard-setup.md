# Cursor Teams dashboard setup (UGA Online admins)

Configure at **[cursor.com/dashboard](https://cursor.com/dashboard) → Rules**.

Team Rules are **free-form text** (not `.mdc` files). Keep canonical sources in this hub's [`.cursor/rules/`](../../.cursor/rules/) and paste here when updating. After quarterly review, sync dashboard content from those files.

**Precedence:** Team Rules → Project Rules → User Rules. Enforced rules cannot be disabled by team members.

---

## Dashboard hygiene (one-time)

1. **Privacy Mode** — Enable org-wide (Teams default; verify under team settings).
2. **SSO/SAML** — Configure if UGA IT requires it.
3. **Invite members** — Share team invite link; each member gets Team Rules automatically.

---

## Enforced Team Rules

Create each rule with **Enforce this rule** checked and **no glob pattern** (applies to every conversation).

### 1. `uga-online-secrets-and-ferpa`

**Enforce:** Yes · **Glob:** (none)

Paste from [`.cursor/rules/uga-online-secrets-and-ferpa.mdc`](../../.cursor/rules/uga-online-secrets-and-ferpa.mdc) (body only, skip YAML frontmatter):

```
# Secrets and FERPA (UGA Online)

## Secrets

- **Never** commit `.env`, `.env.local`, API keys, OAuth client secrets, Kaltura admin secrets, or LLM provider keys.
- Use gitignored env files, CI secrets, or institutional secret stores.
- In PR descriptions and screenshots, **redact** tokens and credentials.
- If a diff adds suspicious patterns (`apiKey`, `Bearer `, private keys), stop and warn the user.

## FERPA

- Do not paste **student names**, **UGA IDs**, or **identifiable grade data** into AI prompts or committed files.
- Aggregated, anonymized data is generally acceptable when policy allows; when in doubt, remove identifying fields.

## PRs

- Flag secrets or PII in diffs during review and pre-PR checks.
- Playbook: `docs/cursor/pr-and-code-review.md`
```

### 2. `uga-online-upstream-repo-tiers`

**Enforce:** Yes · **Glob:** (none)

Paste from [`.cursor/rules/upstream-reference-repos.mdc`](../../.cursor/rules/upstream-reference-repos.mdc) (body only):

```
# UGA Online upstream reference repos

## Repo tiers

| Tier | Folder name | Agent behavior |
|------|-------------|----------------|
| **Strict reference** | `UGA-Brightspace-React-Template` | Do not edit unless the user explicitly targets this repo |
| **Canonical upstream** | `uga-online-brightspace-agent-framework`, `uga-lit-components` | Read-only during app/agent work; no drive-by fixes; upstream changes only via intentional PRs |
| **Work repos** | `UGA-Brightspace-React-Apps`, `agent-todd`, `uga-drive-elc-sync`, `uga-timeline-js-aws`, `uga-elc-kaltura-caption-import` | Normal edits |
| **Docs hub** | `uga-online-cursor-docs` | Rules and team docs only; do not paste full upstream code here |

## Before editing

1. Check the path or workspace folder name against the table above.
2. If the user asks to build or change an **app feature**, implement in a **work repo**. For **new** course-file React apps, you **must** use `UGA-Brightspace-React-Apps/apps/<kebab-case-name>/`.
3. If the change belongs only in a reference or canonical upstream repo, **stop** and tell the user to open an upstream PR to `uga-ool`, or use the **scaffold-from-uga-online-template** skill to copy patterns into a work repo.

## New course-file React apps

- Any **new** Brightspace course-file React app **must** live under **`UGA-Brightspace-React-Apps/apps/<kebab-case-name>/`**.
- Do not create new course-file React apps as sibling repos or only inside `UGA-Brightspace-React-Template`.
```

### 3. `uga-online-design-system`

**Enforce:** Yes · **Glob:** (none — applies to every conversation)

The UGA Online Design System is **always** used for course content, HTML shells, and UI. Paste from [`.cursor/rules/uga-online-design-system.mdc`](../../.cursor/rules/uga-online-design-system.mdc) (body only):

```
# eLC and UGA Online Design System

**Always use the UGA Online Design System** for course content, HTML shells, and UI in eLC (Brightspace) tools unless the user explicitly targets a legacy file that cannot be migrated yet.

**eLC** is UGA’s name for **D2L Brightspace**. Treat Valence paths, session cookies, and Manage Files / Content hosting constraints as first-class.

## Design system (CDN)

Official docs: [Installation](https://design.online.uga.edu/getting-started/installation/).

- **CSS (latest):** `https://design.online.uga.edu/css/base.css`
- **JS:** `https://design.online.uga.edu/js/scripts.js` (before closing `</body>`)
- **Fonts:** load Merriweather, Merriweather Sans, and Oswald from Google Fonts as documented on the installation page (design system expects them).
- **Production:** prefer **versioned** URLs (e.g. `https://design.online.uga.edu/v1.5.3/css/base.css` and matching `.../js/scripts.js`) so LMS pages do not change unexpectedly when the CDN updates.

## HTML / entry templates

- Include a **skip link** and wrap primary content in **`<main id="main-content">`** when using full-page patterns from the design system.
- Use documented layout utilities where appropriate (e.g. `obj-reading-width` from the minimal template on the installation page).
- For **course-file** React apps, ensure the **built** `index.html` still loads design-system links correctly (absolute CDN URLs are fine).

## Brightspace integration

- Respect **`VITE_API_BASE_URL`**, **`VITE_LP_VERSION`**, and **`VITE_LE_VERSION`** (and any app-specific `VITE_*` from the project README).
- Do not assume shadow DOM for UGA Online course-file tools unless the project already does; match existing patterns in the repo.
```

---

## Optional Team Rules (glob-scoped)

Create with **Enforce:** No (members may disable) unless your policy requires otherwise. Set the **glob pattern** as listed.

| Rule name | Glob | Source |
|-----------|------|--------|
| `elc-ai-agent-framework` | `**/frontend/src/agents/**` | [`.cursor/rules/elc-ai-agent-tool.mdc`](../../.cursor/rules/elc-ai-agent-tool.mdc) |
| `uga-react-course-files` | `**/src/**/*.{ts,tsx}` | [`.cursor/rules/uga-brightspace-react-template.mdc`](../../.cursor/rules/uga-brightspace-react-template.mdc) |
| `uga-online-html-cleanup` | `**/*.{html,htm}` | [`.cursor/rules/uga-online-html-cleanup.mdc`](../../.cursor/rules/uga-online-html-cleanup.mdc) |

For each optional rule, paste the markdown body from the source file (skip YAML frontmatter).

---

## Verification checklist

After setup, confirm with one team member:

- [ ] Team Rules appear under **Cursor Settings → Rules → Team Rules**
- [ ] Enforced rules cannot be toggled off
- [ ] Agent applies UGA Online Design System CDN and layout patterns for HTML/UI work (design system rule active)
- [ ] Opening an HTML file in Agent chat picks up `uga-online-html-cleanup` (optional rule)
- [ ] Agent refuses to commit secrets when asked (FERPA rule active)

---

## Quarterly sync procedure

1. Diff each hub `.mdc` file against dashboard rule text.
2. Update dashboard if content drifted.
3. Note changes in [`shared-resources-catalog.md`](shared-resources-catalog.md) maintainer log (optional PR).

Catalog index: [`shared-resources-catalog.md`](shared-resources-catalog.md).
