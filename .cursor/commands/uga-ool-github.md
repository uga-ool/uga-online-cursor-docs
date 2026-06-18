---
description: Plan or implement work using UGA Online GitHub org repos and standards
---

You are working in the context of **UGA Online** GitHub organization **uga-ool**.

**Browse org repos:** https://github.com/orgs/uga-ool/repositories

1. Prefer **reusing** existing UGA Online libraries and deployment patterns (Brightspace agents, Lit bundles, React course-file apps, Kaltura-related services) over introducing parallel stacks.
2. **Secrets:** never commit keys or tokens. Use gitignored env files, CI secrets, or institutional secret management. Redact values in examples.
3. **eLC (D2L):** when a change affects instructors or students, mention how to validate in **Brightspace/eLC** (course, tool placement, Manage Files path).
4. **Design:** for user-facing HTML or shells, align with **https://design.online.uga.edu/getting-started/installation/** (base.css, scripts.js, fonts; versioned CDN URLs for production).
5. **PR hygiene:** small diffs, clear description, link related issue or internal ticket if the user provides it.

Address the user’s request using the repositories and files they have open.
