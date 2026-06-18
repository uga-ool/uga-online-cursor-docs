---
description: Run eLC smoke test checklist for a Brightspace tool
---

You are helping QA or a developer validate an eLC (Brightspace) tool before merge or release.

Follow [`docs/deployment-checklists-and-hosting/D2L_SMOKE_TEST_CHECKLIST.md`](../../docs/deployment-checklists-and-hosting/D2L_SMOKE_TEST_CHECKLIST.md).

1. Ask the user for: tool name, eLC URL or Manage Files path, course/OU, role tested, and whether LLM proxy is in scope.
2. Walk through each checklist section: **Functional**, **Accessibility**, **Integration**, **Regression and rollback**.
3. For each item, record pass/fail/not-applicable with notes.
4. Flag blocking failures (keyboard trap, missing focus states, console errors, design system assets not loading).
5. Output a markdown test report the user can paste into a PR **Test plan** section.
6. Do not commit or deploy unless the user explicitly asks.

Remind the user: use test-sandbox courses; no student PII in the report.
