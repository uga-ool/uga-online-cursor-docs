---
description: User message sent to the LLM when a single-standard review is initiated
used_by: MainView.tsx, StandardsView.tsx → review request prompt
variables: standardSetName, standardId, description
---
Review this course against {{standardSetName}} Standard {{standardId}}: "{{description}}". Use the Brightspace API tools to inspect relevant course elements and provide specific, evidence-based feedback.
