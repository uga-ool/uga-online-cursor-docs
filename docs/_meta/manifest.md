# Document provenance

Use this list when refreshing content from upstream repositories. Paths are relative to this repo’s `docs/` folder unless noted.

| Topic folder (this repo) | Typical upstream repository | Notes |
|--------------------------|----------------------------|--------|
| `d2l-brightspace/` | `uga-lit-components` | Originally `docs/D2L_*.md`, quiz JSON, course template spike. |
| `content-markdown-and-drive-sync/` | `uga-drive-elc-sync`, `uga-lit-components` | Drive sync READMEs, `ELC_GOOGLE_SYNC_*` from lit `docs/`, `ELC_DEPLOY_FUTURE.md`. |
| `deployment-checklists-and-hosting/` | `gamification-widget`, `uga-timeline-js-aws` | LTI, smoke tests, SAM, rollout, migration. |
| `timeline-aws/` | `uga-timeline-js-aws` | API contract, Parquet, archived demo notes (not vendored TimelineJS). |
| `ai-agents-brightspace/` | `uga-online-brightspace-agent-framework-main` | Guides, TODOs, example prompts, pipeline READMEs. |
| `lti-widgets-gamification/` | `gamification-widget` | Handoff + agent prompts. |
| `course-file-react-apps/` | `UGA-Brightspace-React-Apps` | Monorepo + template + app READMEs. |
| `lit-component-library/` | `uga-lit-components` | Library README, CHANGELOG, improvement notes, server READMEs, Copilot instructions. |
| `captions-and-transcripts/` | `uga-elc-kaltura-caption-import` | Handoff + overview. |
| `pdf-accessibility/` | `pdf-remediation-designer-guide` | Static HTML references. |

Forks such as `agent-todd` and `thestater-agent` generally match the framework; refresh from `uga-online-brightspace-agent-framework-main` unless you maintain fork-specific diffs elsewhere.
