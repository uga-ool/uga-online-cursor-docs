# Cursor chat and transcript practices (UGA Online)

Standards for Cursor IDE chat sessions across instructional design, QA, development, and sysadmin roles.

**Not the same as:** Kaltura/video transcripts ([`docs/captions-and-transcripts/`](../captions-and-transcripts/)) or eLC agent conversation transcripts ([`TODO-CORE-GAPS.md`](../ai-agents-brightspace/planning/TODO-CORE-GAPS.md)).

---

## When to start a new chat

Start a **new** Cursor chat when:

- You begin a new feature, bug fix, or review task (not a small follow-up on the same branch)
- Context has accumulated ~30+ minutes of unrelated exploration
- You hand off between roles (ID → developer, developer → QA)
- You switch repos (hub → template repo) for implementation work

**Continue the same chat** when:

- Iterating on the same file or PR on the same branch
- Answering reviewer comments on a change you just made
- Short clarifying questions about the current task

---

## Handoff protocol

Use when passing work between teammates or between repos.

### Handoff Summary template

Copy this block into a **new** chat in the target repo. Do **not** paste the full prior transcript.

```markdown
## Handoff Summary

**Goal:**
**Repos / paths:**
**Files touched or to create:**
**Decisions already made:**
**Open questions:**
**eLC test context:** (OU, role, Manage Files path — or N/A)
**Out of scope:**
**FERPA note:** (confirm no student PII in this handoff)
```

### Steps

1. Author fills the Handoff Summary (or uses [`prompts/handoff-new-project.md`](prompts/handoff-new-project.md)).
2. Open a new chat in the **target repo** (the repo where commits will land).
3. `@`-reference relevant docs (e.g. `@AGENT-GUIDE.md`, `@docs/cursor/pr-and-code-review.md`).
4. Paste the Handoff Summary as the first message.
5. Durable outcomes → commit to `docs/` or PR description, not chat history.

Example handoff pattern: [`docs/captions-and-transcripts/COURSE_TRANSCRIPTS_TO_FILES_HANDOFF.md`](../captions-and-transcripts/COURSE_TRANSCRIPTS_TO_FILES_HANDOFF.md).

---

## FERPA-safe practices

- **Never** paste student names, UGA IDs, grade data, or course rosters into Cursor chat
- Use anonymized placeholders: `Student A`, `OU: sandbox-dev-001`
- Before sharing a chat export: redact tokens, session URLs, API keys, and any PII
- Prefer linking to committed handoff docs over sharing raw chat exports
- Aggregated, anonymized data is acceptable when institutional policy allows; when in doubt, remove identifying fields

See also: [`.cursor/rules/uga-online-secrets-and-ferpa.mdc`](../../.cursor/rules/uga-online-secrets-and-ferpa.mdc).

---

## Archiving and durable artifacts

| Outcome type | Where it lives |
|--------------|----------------|
| Code changes | Git commit in the correct work repo |
| Test plan / QA results | PR description or deployment checklist |
| Agent prompt design | `docs/ai-agents-brightspace/example-prompts/` (eLC agents) or repo `prompts/` |
| Process decisions | `docs/cursor/` or team meeting notes |
| Ephemeral exploration | Discard — no team archive of raw chats required |

Cursor Teams **shared chats** may be used for specific collaborative workflows; default is individual chats with handoff summaries.

---

## Mode guidance by task

| Task | Mode |
|------|------|
| Explore docs, draft PR summary, review HTML | **Ask** |
| Implement features, apply HTML cleanup | **Agent** (review every diff) |
| Large or risky changes | **Plan** first, then Agent |
| PR review | **Ask** or subagent `uga-online-pr-reviewer` |

---

## Prompt starters

Reusable first messages: [`docs/cursor/prompts/`](prompts/).

Role quick-starts: [`docs/cursor/roles/`](roles/).

Catalog: [`shared-resources-catalog.md`](shared-resources-catalog.md).
