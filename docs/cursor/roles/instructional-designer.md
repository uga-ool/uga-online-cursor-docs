# Instructional designer — Cursor quick-start

~15 min. For HTML remediation, course content review, and eLC agent feature requests.

## Your workspace

Copy [`UGA-Online-InstructionalDesign.code-workspace`](../../workspaces/examples/UGA-Online-InstructionalDesign.code-workspace) and fix folder paths. Or open **File → Open Folder** → `uga-online-cursor-docs` plus your content repo.

## Cursor modes

| Mode | Use for |
|------|---------|
| **Ask** | Review HTML, draft agent requests, explore docs — no file edits |
| **Agent** | Apply HTML cleanup fixes, edit content files — review every diff |
| **Plan** | Large remediation batches or new workflow design before Agent edits |

## Key commands

| Command | When |
|---------|------|
| `html-cleanup` | Remediate HTML from PDF tools or ID Assistant (includes required UGA Online Design System) |
| `agent-request` | Request a new eLC AI agent from a developer |
| `elc-design-system` | Apply UGA Online Design System to HTML shells (also enforced via `uga-online-design-system` Team Rule) |

Subagent: **`uga-online-html-reviewer`** — accessibility and semantic HTML review.

## Key docs

- HTML cleanup rule (includes required design system): [`.cursor/rules/uga-online-html-cleanup.mdc`](../../.cursor/rules/uga-online-html-cleanup.mdc)
- Design system rule: [`.cursor/rules/uga-online-design-system.mdc`](../../.cursor/rules/uga-online-design-system.mdc)
- PDF accessibility reference: [`docs/pdf-accessibility/`](../pdf-accessibility/)
- Agent request template: [`docs/ai-agents-brightspace/guides/AGENT-REQUEST-TEMPLATE.md`](../ai-agents-brightspace/guides/AGENT-REQUEST-TEMPLATE.md)
- Design system: [design.online.uga.edu](https://design.online.uga.edu/getting-started/installation/)
- Chat handoffs: [`chat-and-transcript-practices.md`](chat-and-transcript-practices.md)
- Prompt starters: [`prompts/html-remediation-review.md`](prompts/html-remediation-review.md), [`prompts/agent-feature-request.md`](prompts/agent-feature-request.md)

## FERPA reminder

Do not paste student names, UGA IDs, or grade data into Cursor chat. Use sandbox course OUs and anonymized examples.

---

**Open this workspace** → **Run `html-cleanup`** → **Read [`chat-and-transcript-practices.md`](chat-and-transcript-practices.md)**
