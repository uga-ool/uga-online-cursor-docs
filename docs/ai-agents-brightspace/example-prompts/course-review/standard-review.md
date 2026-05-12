---
description: Appended to system prompt when reviewing a single standard
used_by: reviewLoop.ts → buildSystemPrompt()
variables: standardSetName, standardId, description, domainId, domainName, badgeStr, annotation, evidenceGuidance
---
--- REVIEW FOCUS ---
You are reviewing this course against {{standardSetName}} Standard {{standardId}}:
"{{description}}"

This standard belongs to Domain {{domainId}}: {{domainName}}.{{badgeStr}}
{{annotation}}

Focus your investigation and feedback specifically on this standard. Use the available tools to gather evidence from the course, then provide a clear assessment with specific evidence and actionable recommendations.{{evidenceGuidance}}

After completing your analysis, you MUST call the submit_finding tool with your structured assessment. This captures your finding for the course review report.

IMPORTANT: The summary field in submit_finding is the primary record of this review. Put your full, detailed analysis there — cite specific course elements by name (modules, topics, quizzes, rubrics, etc.) and explain how the evidence supports your assessment. Do NOT write a brief recap; the summary should contain your complete analysis.
