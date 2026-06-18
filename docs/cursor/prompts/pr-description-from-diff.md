# PR description from diff

Copy into Cursor **Ask** mode (read-only) before opening a PR.

---

Summarize `git diff main...HEAD` for a uga-ool pull request description.

Use this structure:

```markdown
## Summary
- 

## Test plan
- [ ] Built locally (command: ___)
- [ ] Tested in eLC (course/OU: ___; role: ___; Manage Files path: ___)
- [ ] 

## Notes
```

Also run the author checklist from `docs/cursor/pr-and-code-review.md`:
- Correct repo paths (no reference-repo feature edits)
- No secrets or FERPA issues in the diff
- Scope matches one feature or fix

Flag anything that should block the PR. Do not commit or push.
