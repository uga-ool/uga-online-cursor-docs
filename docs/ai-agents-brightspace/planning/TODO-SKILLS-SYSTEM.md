# TODO: Skills System

## Context

Skills are packages of instructions, examples, reference materials, and optionally executable functions that agents can discover and selectively load into context. They follow a **progressive disclosure** model — the agent sees only skill names and descriptions by default, loads the full `SKILL.md` when relevant, and can request additional linked files or activate skill-provided tools as needed.

This design is informed by [Anthropic's Agent Skills pattern](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills): skills as directories with a `SKILL.md` entry point, YAML frontmatter for metadata, and linked resources for deeper context.

The system has three tiers:

| Tier | Scope | Stored Where | Managed By | Can Include JS? |
|------|-------|-------------|------------|-----------------|
| Framework | All agents, all deployments | Codebase (`frontend/src/skills/`) | Developer | Yes (bundled at build time) |
| Program/Course | All users in that org unit | Course files (`{agent}/skills/`) | Program coordinator | No (markdown + declarative composition only) |
| Personal | One user only | Course files (`{agent}/skills/user_{userId}/`) | The user | No (markdown only) |

**Priority**: Personal > Program > Framework (when skills conflict).

## What Exists Today

- Prompt markdown files with frontmatter (`?raw` imports, `parseFrontmatter`, `interpolate`)
- `import.meta.glob` for build-time file discovery (used by `catalog.ts` for agents, prompts, data files)
- Dev shell with tabs for Tools, Prompts, Services, Providers, Schema, Framework Tools
- `/__dev` API for reading/writing agent config files during development
- Tool Editor pattern: `data/tools.json` with `disabledTools` array, toggled via dev UI
- `readCourseFile` / `listCourseFolder` for runtime file discovery
- Course-embeddings pipeline for semantic search (if skill library grows large)

---

## Skill Format

### `SKILL.md` — the entry point

Every skill directory contains a `SKILL.md` file. This replaces the previous `index.json` manifest — a single file serves as both metadata and primary instructions.

```markdown
---
name: Course Accessibility Review
description: Evaluate course content for WCAG 2.1 AA compliance.
version: "1.0"
keywords: [accessibility, WCAG, ADA, alt text, captions]
files:
  - path: checklist.md
    description: WCAG 2.1 AA checklist for course content
  - path: examples/good-alt-text.md
    description: Examples of effective alt text
tools:
  - name: extract_images_without_alt
    module: ./extractImages.ts
    description: Parse HTML content and return all images missing alt text.
  - name: evaluate_alt_text_quality
    module: ./evaluateAltText.ts
    description: Score existing alt text against WCAG criteria and suggest improvements.
---

## Use Case: Full Accessibility Audit
Trigger: User asks to "review accessibility", "check ADA compliance", or "audit for WCAG"
Steps:
1. Load the WCAG checklist (checklist.md)
2. Fetch the course table of contents
3. For each content topic, use `extract_images_without_alt` to find issues
4. Use `evaluate_alt_text_quality` on images that have alt text
5. Generate a findings report grouped by severity (critical, major, minor)
6. Suggest specific remediation steps for each finding
Result: Structured accessibility audit report with actionable remediation items

## Use Case: Quick Alt Text Check
Trigger: User asks about "alt text", "image descriptions", or "image accessibility"
Steps:
1. Load alt text examples (examples/good-alt-text.md)
2. Fetch the specified content page or module
3. Run `extract_images_without_alt` on the page content
4. Suggest improvements for missing or inadequate alt text
Result: List of images with alt text assessment and suggested improvements

## General Instructions

When performing accessibility reviews, always reference specific WCAG 2.1 AA
success criteria by number (e.g., SC 1.1.1 Non-text Content). Prioritize
findings by impact on learner experience.

Focus on the most common accessibility issues in course content:
- Images without alt text or with decorative images not marked as such
- Videos without captions or transcripts
- PDFs that are not tagged for screen readers
- Color contrast issues in custom HTML content
- Tables without proper header markup
```

### Key design principles

**YAML frontmatter** — machine-readable metadata that goes into the catalog:
- `name` and `description`: What the agent sees in every conversation
- `keywords`: For catalog search and potential semantic matching
- `files`: Linked resources the agent can request via `load_skill` (progressive disclosure)
- `tools`: Functions the skill provides (Tier 1 only — see "Executable Functions" below)

**Use case definitions** — structured blocks that tell the agent exactly when and how to use the skill:
- `Trigger`: Concrete phrases or situations that should activate this skill
- `Steps`: The procedural workflow to follow (may reference tools, linked files, or framework tools)
- `Result`: The expected outcome (gives the agent a success criterion)

Multiple use cases per skill are encouraged when a skill covers related but distinct workflows.

**General instructions** — the detailed procedural knowledge, best practices, and domain context. This is the "onboarding guide" content.

### Progressive disclosure levels

1. **Catalog** (always in context): `name` + `description` from frontmatter (~50-100 tokens per skill)
2. **SKILL.md body** (loaded on demand): Use case definitions + general instructions
3. **Linked files** (loaded on demand): Checklists, examples, reference material via `load_skill(id, files)`
4. **Skill tools** (registered on demand): Functions activated when the skill is loaded

---

## Executable Functions in Skills

### Tier 1: Bundled TypeScript (Framework Skills)

Framework skills can include TypeScript/JavaScript modules alongside their markdown. These are bundled at build time, reviewed by developers, and registered as tools when the skill is loaded.

**Why**: Some operations are better handled by deterministic code than token-by-token LLM generation — parsing HTML, extracting structured data, running validation rules, formatting output, computing scores.

**Directory structure**:
```
frontend/src/skills/accessibility-review/
├── SKILL.md
├── checklist.md
├── extractImages.ts          ← exported function
├── evaluateAltText.ts        ← exported function
└── examples/
    └── good-alt-text.md
```

**Function contract**: Each tool module exports a single function with typed parameters and return value:

```typescript
// extractImages.ts
export interface ExtractImagesParams {
  html: string;
}

export interface ImageInfo {
  src: string;
  alt: string | null;
  context: string;        // surrounding text for context
  hasAlt: boolean;
}

export default function extractImagesWithoutAlt(
  params: ExtractImagesParams
): ImageInfo[] {
  // deterministic HTML parsing — no LLM needed
  const parser = new DOMParser();
  const doc = parser.parseFromString(params.html, 'text/html');
  const images = doc.querySelectorAll('img');

  return Array.from(images)
    .filter(img => !img.alt || img.alt.trim() === '')
    .map(img => ({
      src: img.src,
      alt: img.alt || null,
      context: img.parentElement?.textContent?.slice(0, 100) ?? '',
      hasAlt: false,
    }));
}
```

**Registration flow**:
1. Agent loads skill catalog at startup (frontmatter only)
2. LLM calls `load_skill("accessibility-review")`
3. Framework reads `SKILL.md` body and injects into context
4. Framework checks `tools` in frontmatter → dynamically imports the modules
5. Skill functions are registered as available tools for the remainder of the conversation
6. LLM can now call `extract_images_without_alt({ html: "..." })` like any other tool

**Security**: Safe — these are developer-authored TypeScript compiled into the bundle. Same trust level as any other framework code.

### Tier 2: Declarative Tool Composition (Program Skills) — Future

Program coordinators can't write arbitrary JS, but they can define new operations by composing existing framework tools and skill functions in YAML:

```yaml
tools:
  - name: audit_module_images
    description: Fetch all topics in a module and report images without alt text.
    parameters:
      moduleId: { type: string, description: "The module to audit" }
    compose:
      - call: get_module_structure
        with: { moduleId: "$moduleId" }
        store: topics
      - for_each: $topics.Topics
        call: get_topic_content
        with: { topicId: "$item.TopicId" }
        collect: pages
      - call: extract_images_without_alt
        with: { html: "$pages" }
```

This is a constrained workflow DSL that only calls tools already registered in the framework. No arbitrary code execution.

**When to implement**: Defer until there's a clear pattern of program coordinators wanting multi-step operations that the LLM handles inconsistently on its own.

### Tier 3: Markdown Only (Personal Skills)

Personal skills are instructions and context only. No executable functions, no composition. The LLM follows the instructions using whatever tools are already available.

**Rationale**: Personal skills are user-authored with minimal review. Keeping them markdown-only eliminates any code execution risk and keeps the authoring experience simple.

---

## Work Items

### 1. SKILL.md Schema and Parser

Parse `SKILL.md` frontmatter into a typed manifest:

```typescript
// core/skills/types.ts

export interface SkillManifest {
  id: string;                           // derived from directory name
  name: string;
  description: string;
  version?: string;
  keywords?: string[];
  tier: 'framework' | 'program' | 'personal';
  files?: SkillFileRef[];
  tools?: SkillToolRef[];
}

export interface SkillFileRef {
  path: string;
  description: string;
}

export interface SkillToolRef {
  name: string;
  module: string;                       // relative path to TS/JS module
  description: string;
}

export interface SkillUseCaseDefinition {
  name: string;                         // parsed from "## Use Case: <name>"
  trigger: string;
  steps: string[];
  result: string;
}
```

- [ ] Create `core/skills/types.ts` with manifest and use case types
- [ ] Create `core/skills/parser.ts` — parse `SKILL.md` frontmatter via existing `parseFrontmatter`
- [ ] Extract use case definitions from the markdown body (parse `## Use Case:` sections)
- [ ] Validate required fields (`name`, `description`) with Zod
- [ ] Add types to `shared/src/types/` and export from `shared/src/index.ts`

### 2. Framework-Level Skills Library (Tier 1)

**Directory structure**:
```
frontend/src/skills/
├── accessibility-review/
│   ├── SKILL.md
│   ├── checklist.md
│   ├── extractImages.ts
│   ├── evaluateAltText.ts
│   └── examples/
│       └── good-alt-text.md
├── rubric-alignment/
│   ├── SKILL.md
│   └── alignmentScorer.ts
└── ...
```

**Enablement**: Each agent has a `data/skills.json` config:
```json
{
  "enabledSkills": ["accessibility-review", "rubric-alignment"]
}
```

**Discovery**: Use `import.meta.glob('../skills/*/SKILL.md', { as: 'raw' })` to find all framework skills at build time. Parse frontmatter from each to build the catalog. Only enabled skills are included.

**Dev UI**: Add a "Skills" tab to the dev shell (following the Tool Editor toggle pattern). Lists all framework skills with enable/disable toggles per agent. Writes to `data/skills.json`.

- [ ] Create `frontend/src/skills/` directory convention
- [ ] `import.meta.glob` discovery of `SKILL.md` files
- [ ] Parse frontmatter to build framework skill catalog
- [ ] `data/skills.json` per-agent enablement config
- [ ] Skills tab in dev shell (toggle framework skills per agent)

### 3. Skill Tool Registration (Tier 1 Executable Functions)

When a framework skill is loaded and its frontmatter declares `tools`, dynamically import the referenced modules and register them as available LLM tools.

```typescript
// core/skills/toolLoader.ts

export async function loadSkillTools(
  skillId: string,
  toolRefs: SkillToolRef[]
): Promise<ToolDefinition[]> {
  const modules = import.meta.glob('../skills/*/**.ts');

  const tools: ToolDefinition[] = [];
  for (const ref of toolRefs) {
    const modulePath = `../skills/${skillId}/${ref.module}`;
    const mod = await modules[modulePath]();
    tools.push({
      name: ref.name,
      description: ref.description,
      parameters: extractParametersFromModule(mod),
      execute: mod.default,
    });
  }
  return tools;
}
```

- [ ] Create `core/skills/toolLoader.ts` — dynamic import and registration
- [ ] Define the function contract (default export, typed params/return)
- [ ] Integrate with the tool executor — skill tools are called the same way as framework tools
- [ ] Parameter schema extraction: derive JSON schema from TypeScript interfaces (or require explicit schema export)
- [ ] Tools are registered per-conversation, not globally — they appear only after `load_skill`
- [ ] Unregister skill tools if the skill is unloaded or the conversation resets

### 4. `load_skill` Tool

The LLM calls this to load a skill's content and optionally its linked files:

```
load_skill(skillId: "accessibility-review")
load_skill(skillId: "accessibility-review", files: ["checklist.md"])
```

Behavior:
1. Read `SKILL.md` body (everything below frontmatter) and inject into conversation context
2. If `tools` are declared in frontmatter → dynamically import and register them (Tier 1 only)
3. If `files` parameter is provided → load those specific linked files and include their content
4. If `files` is omitted → load only `SKILL.md` body (the agent can request files later)

Resolution by tier:
- Framework skills: resolve via dynamic `import()` with `?raw` for markdown, standard import for TS modules
- Program/personal skills: resolve via `readCourseFile` for markdown (no tool registration)
- The LLM does not need to know which tier the skill came from

- [ ] Create `core/skills/loadSkill.ts` — resolution logic
- [ ] Register `load_skill` as a framework-level tool (available to all agents with skills enabled)
- [ ] Handle Tier 1 tool registration on load
- [ ] Return structured response: `{ content: string, toolsRegistered?: string[] }`
- [ ] Track loaded skills per conversation to avoid duplicate loading

### 5. Program/Course-Level Skills (Tier 2)

**Directory structure** (in course files):
```
{agent}/skills/
├── uga-review-standards/
│   ├── SKILL.md
│   ├── qm-notes.md
│   └── uga-specific-criteria.md
├── program-onboarding/
│   ├── SKILL.md
│   └── instructions.md
└── user_98765/              ← personal skills (Tier 3, see below)
    └── ...
```

**Discovery at runtime**:
1. `listCourseFolder(ou, '{agent}/skills')` — list all subfolders
2. Filter out `user_*` prefixed folders (those are Tier 3)
3. For each remaining folder, `readCourseFile(ou, '{agent}/skills/{folder}/SKILL.md')` — parse frontmatter only for catalog
4. Merge with framework skill manifests into unified catalog

**Management UI**: An admin panel in the agent where program coordinators can:
- Upload new skill folders (SKILL.md + supporting markdown files)
- Edit existing skill markdown via a textarea
- Enable/disable program skills
- View which skills are active and their source tier

- [ ] Runtime discovery via `listCourseFolder` + `readCourseFile` for `SKILL.md` files
- [ ] Parse frontmatter from runtime skills into `SkillManifest`
- [ ] Merge runtime manifests with framework manifests into unified catalog
- [ ] Ignore `tools` field in Tier 2 frontmatter (log a warning if present)
- [ ] Admin UI for uploading/editing program skill packages

### 6. Personal Skills (Tier 3)

**Directory structure** (in course files):
```
{agent}/skills/user_{whoAmI.Identifier}/
├── my-review-style/
│   ├── SKILL.md
│   └── instructions.md
└── dept-contacts/
    ├── SKILL.md
    └── contacts.md
```

**Discovery**: Same as Tier 2, but scoped to `{agent}/skills/user_{userId}/`. Each user only sees their own personal skills.

**Creation methods**:
- **UI**: User creates a skill through a form in the agent settings (provides name, description, and instruction text → framework generates `SKILL.md`)
- **Agent tool (`save_skill`)**: The LLM can create a personal skill from conversation. E.g., user says "When I ask you to draft feedback, always use a constructive tone." Agent calls `save_skill` to persist this as a personal `SKILL.md`.
- **Editing**: User can view and edit their personal skills in the agent settings

- [ ] Per-user skill folder convention (`{agent}/skills/user_{userId}/`)
- [ ] `save_skill` tool for LLM-driven personal skill creation
- [ ] Generate `SKILL.md` from `save_skill` parameters (name, description, instructions)
- [ ] User-facing settings panel for managing personal skills
- [ ] Skill editing (read `SKILL.md`, present in textarea, write back)

### 7. Skill Catalog in System Prompt

On each conversation, include the unified catalog (frontmatter metadata only, not full content):

```
Available skills (use load_skill to activate when relevant):

FRAMEWORK:
- accessibility-review: Evaluate course content for WCAG 2.1 AA compliance.
  Triggers: "review accessibility", "check ADA compliance", "audit for WCAG"
  Tools: extract_images_without_alt, evaluate_alt_text_quality
- rubric-alignment: Analyze alignment between learning outcomes and assessment rubrics.
  Triggers: "evaluate assessment design", "check rubric alignment"
  Tools: alignment_scorer

PROGRAM:
- uga-review-standards: UGA-specific quality criteria beyond standard rubrics.
  Triggers: "UGA review", "institutional standards"

PERSONAL:
- my-review-style: Personal review style and focus areas.
  Triggers: "draft feedback", "write review"
```

This is small enough (~50-100 tokens per skill) to always include, even with 20-30 skills. The `Triggers` line is extracted from use case definitions in the `SKILL.md` body.

- [ ] Build catalog from unified manifests (framework + program + personal)
- [ ] Extract trigger summaries from use case definitions for catalog display
- [ ] Include tool names in catalog entries (so the LLM knows what functions become available)
- [ ] Precedence: Personal > Program > Framework
- [ ] Format for system prompt injection

### 8. User-Facing Skills Panel

A panel in the agent's settings (not the dev shell) where users can:
- **View** all active skills grouped by tier (Framework / Program / Personal)
- **Toggle** individual skills on/off for their session
- **Create/edit** personal skills
- **See** skill descriptions, use cases, and available tools

- [ ] Skills panel component with tier groupings
- [ ] Per-skill toggle (enable/disable without deleting)
- [ ] Skill detail view showing use cases and tools
- [ ] Personal skill create/edit form
- [ ] Use UGA design system components

### 9. Skills Editing

Both program-level and personal skills should be editable at runtime:
- Read `SKILL.md` and linked files via `readCourseFile`
- Present in a textarea for editing
- Write back via `uploadFileToCourse`
- Framework skills are read-only in production (editable only via codebase / dev shell)

- [ ] Markdown editor component for skill files
- [ ] Read/write via course file APIs
- [ ] Frontmatter preservation on edit (don't lose metadata when editing body)
- [ ] Linked file management (add/remove/rename)

---

## Declarative Tool Composition (Future — Tier 2)

When program coordinators consistently need to define multi-step operations that the LLM handles inconsistently on its own, add a declarative composition format to the `tools` field in Tier 2 `SKILL.md` files:

```yaml
tools:
  - name: audit_module_images
    description: Fetch all topics in a module and report images without alt text.
    parameters:
      moduleId: { type: string, description: "The module to audit" }
    compose:
      - call: get_module_structure
        with: { moduleId: "$moduleId" }
        store: topics
      - for_each: $topics.Topics
        call: get_topic_content
        with: { topicId: "$item.TopicId" }
        collect: pages
      - call: extract_images_without_alt
        with: { html: "$pages" }
```

This is a constrained workflow DSL:
- Only calls tools already registered in the framework or loaded from Tier 1 skills
- No arbitrary code execution
- Variable interpolation from parameters and previous step results
- `for_each` for iteration, `store`/`collect` for accumulation

**When to implement**: Defer until there is a demonstrated need. Start with the LLM following procedural steps in markdown — the use case definition format often provides enough structure for the LLM to execute multi-step workflows reliably. Add declarative composition only when consistency requires it.

### Work Items (deferred)
- [ ] Define the composition DSL schema (Zod validation)
- [ ] Build a composition executor that resolves `$variable` references and `for_each` loops
- [ ] Integrate composed tools into the tool executor alongside native tools
- [ ] Validate that all `call` references point to registered tools (fail-fast on unknown tools)
- [ ] Security review: ensure composition can't be used to exfiltrate data or bypass permissions

---

## Design Decisions to Make

- [ ] Should the `load_skill` tool be a framework-level tool (available to all agents) or agent-defined? *(Recommendation: framework-level, enabled per-agent via skills config)*
- [ ] Should skills support versioning? (If a framework skill is updated, should the agent note the change?)
- [ ] Should there be a maximum number of skills loadable per conversation turn (to manage context window)?
- [ ] Should the agent auto-load personal skills at conversation start, or always wait for explicit loading?
- [ ] At what library size should skill discovery use embeddings instead of keyword matching? *(See `TODO-IMPLEMENTATION-ORDER.md` Phase 6)*
- [ ] Should program coordinators be able to create "recommended" personal skills that users can opt into?
- [ ] How should skill tool parameter schemas be defined — TypeScript interface extraction, explicit JSON schema export, or Zod schema co-located with the function?
- [ ] Should the skill tool loader validate that function signatures match declared parameters at build time?
- [ ] For the composition DSL: should `for_each` support concurrency (parallel tool calls) or only sequential execution?
