# Systems administrator — Cursor quick-start

~15 min. For deployment, OAuth/LTI, branch protection, and secrets governance.

## Your workspace

Copy [`UGA-Online-Develop.code-workspace`](../../workspaces/examples/UGA-Online-Develop.code-workspace) or open **File → Open Folder** → `uga-online-cursor-docs` for runbooks and checklists.

## Cursor modes

| Mode | Use for |
|------|---------|
| **Ask** | Explore deployment docs, draft runbooks, review config diffs |
| **Agent** | Update deployment markdown, SAM templates — review every diff |
| **Plan** | Infrastructure or rollout changes before Agent edits |

## Admin responsibilities

| Area | Doc / action |
|------|--------------|
| **Cursor Teams dashboard** | Configure Team Rules per [`team-rules-dashboard-setup.md`](team-rules-dashboard-setup.md) |
| **Branch protection** | Require **Validate commit messages** on `main` — [`pr-and-code-review.md`](pr-and-code-review.md) |
| **Privacy Mode** | Verify org-wide in Cursor dashboard |
| **SSO/SAML** | Configure with UGA IT if required |

## Key docs

- OAuth and auth: [`docs/d2l-brightspace/`](../d2l-brightspace/)
- LTI and deploy: [`docs/deployment-checklists-and-hosting/`](../deployment-checklists-and-hosting/)
- Production hardening: same folder (`PRODUCTION_HARDENING`, `SAM`, rollout docs)
- Secrets: [`.cursor/rules/uga-online-secrets-and-ferpa.mdc`](../../.cursor/rules/uga-online-secrets-and-ferpa.mdc)
- Shared resource catalog: [`shared-resources-catalog.md`](shared-resources-catalog.md)

## Secrets (never in git)

- `.env`, OAuth client secrets, Kaltura admin secrets, LLM keys → gitignored env files, CI secrets, or institutional secret stores
- Flag suspicious patterns in PR review: `apiKey`, `Bearer `, private keys

## eLC validation context

When reviewing deploy PRs, confirm authors included OU, role, Manage Files path, and rollback URL in the test plan.

---

**Open hub repo** → **Configure Team Rules in dashboard** → **Read [`team-rules-dashboard-setup.md`](team-rules-dashboard-setup.md)**
