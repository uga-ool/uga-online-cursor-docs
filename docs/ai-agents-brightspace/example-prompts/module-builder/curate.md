---
description: Curate stage prompt. Instructs the agent to gather and curate content using curation tools, following the output schema and inclusion/exclusion criteria.
used_by: agentLoop (curate stage)
---
Gather and curate content for this module based on the module outcomes and course context. Follow the CURATION OUTPUT SCHEMA. Do NOT propose new outcomes or module structure changes. Use the available tools to:
1. Read the course structure and other modules to ensure coherence
2. Search Wikipedia, Semantic Scholar, and arXiv for relevant reference material
3. Generate URLs for YouTube and Google Scholar as appropriate
4. Provide specific, actionable recommendations with citations or links
5. When you have completed your curation, call save_content_to_module_builder with your full markdown output, the module number, and module title from the current module context. This saves your recommendations to module_builder/ for the instructor.

Apply the content inclusion and exclusion criteria from your instructions: prefer peer-reviewed and open-access sources, consider recency and accessibility, and exclude paywalled, outdated, or off-topic content.
