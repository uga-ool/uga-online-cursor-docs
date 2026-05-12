# D2L Smoke Test Checklist

## Functional
- [ ] Open tool from D2L and confirm initial load succeeds.
- [ ] Select artifact and complete all required fields.
- [ ] Confirm generation remains blocked until all checklist items are completed.
- [ ] Run generation and confirm output appears.
- [ ] Copy prompt/output and download prompt/output.

## Accessibility
- [ ] Complete full workflow with keyboard only.
- [ ] Focus states are visible on interactive controls.
- [ ] Status updates are announced and visible.

## Integration
- [ ] Design system assets load (`base.css`, scripts).
- [ ] Proxy LLM call succeeds (`/api/llm/chat`), and stream endpoint if configured.
- [ ] No console errors during normal flow.

## Regression and rollback
- [ ] Compare behavior against legacy `Vibe Coding Widget.html` for required guardrails.
- [ ] Rollback launch URL tested and documented.
