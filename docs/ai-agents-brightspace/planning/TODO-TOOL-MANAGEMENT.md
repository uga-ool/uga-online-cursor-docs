# TODO: Dynamic Tool Management

## Context

The framework currently has ~94 tools across 10 categories (74 Brightspace built-ins + 20 service tools). With the default disabled list, agents still see ~70 tools, consuming 3,500-7,000 tokens of context on every LLM call. As agents gain skills tools (`load_skill`, `save_skill`), journal tools (`save_note`, `search_notes`), and cross-course tools, the tool count will grow further.

Large tool sets cause three problems:
1. **Token overhead** — tool definitions are sent on every request, consuming context window budget
2. **Selection confusion** — LLMs make worse tool choices with many similar options
3. **Latency** — larger request payloads, slower responses

## What Exists Today

- **Category filtering** — agents declare which categories they need in their workflow config (`tools: ['content', 'course', ...]`). Static, set at build time.
- **Disabled tools list** — `data/tools.json` with `disabledTools` array, toggled via dev UI Tool Editor. Static per agent.
- **Allowlist option** — `getToolDefinitionsForLLM` in `adapters.ts` already supports an `allowlist` parameter that overrides category-based selection.
- **`resolveUrl` already supports `args.ou`** — tools can target different org units without needing separate tool definitions.

### Current tool inventory

| Category | Tools | Default disabled |
|----------|-------|-----------------|
| content | 15 | 6 |
| course | 12 | 5 |
| discussions | 8 | 3 |
| grades | 12 | 6 |
| quizzes | 6 | 2 |
| dropbox | 6 | 2 |
| rubrics | 5 | 0 |
| other | 10 | 0 |
| Asana (service) | 16 | 0 |
| Google Drive (service) | 4 | 0 |

## Proposed Approach: Tiered Tool Loading

A hybrid between static category filtering and the Anthropic Tool Search pattern. Rather than free-text tool search, use the existing category system made dynamic within a conversation.

### Tier 1: Always-Present Core Tools (~5-10 tools)

A small set of tools the agent needs on every turn. Defined per agent. For a conversational agent like the ID Assistant:

```
save_note, search_notes, load_skill, get_content_toc, read_course_file, get_manage_files, activate_tools
```

For a pipeline agent like course-review, the core set might be different (or the agent might activate all categories at the start and keep them).

These are always included in the `tools` array sent to the LLM. Token cost: ~500-1,000 tokens.

### Tier 2: On-Demand Category Activation

The LLM calls a meta-tool to bring additional tool categories into scope:

```
activate_tools(categories: ["grades", "discussions"])
```

The system prompt describes available categories with a one-line summary:

```
Additional tool categories (call activate_tools to enable):
- content (15 tools): Course content, modules, topics, file reading
- grades (12 tools): Grade objects, categories, schemes, statistics
- discussions (8 tools): Forums, topics, posts
- quizzes (6 tools): Quiz definitions, questions
- dropbox (6 tools): Assignments, submissions, feedback
- rubrics (5 tools): Rubric definitions, criteria, levels
- other (10 tools): News, calendar, checklists, surveys, LTI
```

On the next turn after `activate_tools`, the LLM sees the core tools plus the activated categories. Categories stay active for the rest of the conversation (or until deactivated).

### Tier 3: Full Tool Search (Future, If Needed)

If the tool count grows beyond ~150 or categories become too coarse, implement a `search_tools(query)` meta-tool that uses keyword or semantic matching against tool names and descriptions. This is the Anthropic pattern. Not needed at current scale.

## Work Items

### 1. Define Core Tool Sets

Each agent declares a `coreTools` list in its config — the tools always present in every LLM call.

```typescript
export const agentConfig = {
  id: 'id-assistant',
  // ...
  coreTools: ['save_note', 'search_notes', 'load_skill', 'get_content_toc', 'read_course_file', 'get_manage_files'],
};
```

Pipeline agents like course-review might set `coreTools: 'all'` to keep current behavior (all enabled tools always present).

### 2. `activate_tools` Meta-Tool

A framework-level tool that expands the active tool set for subsequent turns.

```typescript
{
  name: 'activate_tools',
  description: 'Load additional tool categories for use in this conversation.',
  params: [
    { name: 'categories', type: 'array', description: 'Tool categories to activate', required: true }
  ]
}
```

Returns: Confirmation message listing the newly available tools by name, so the LLM knows what it just gained access to.

Optional companion: `deactivate_tools(categories)` to release categories no longer needed and reduce token overhead on subsequent turns.

### 3. Active Tool State in Conversation

The agent loop needs to track which categories are active across turns. This is conversation state, not persistent state.

```typescript
interface ConversationToolState {
  coreTools: string[];           // Always present
  activeCategories: string[];    // Activated during conversation
}
```

On each LLM call, build the tool list:
1. Start with `coreTools`
2. Add all tools from `activeCategories`
3. Filter out `disabledTools`
4. Convert via `getToolDefinitionsForLLM`

### 4. Update `getToolDefinitionsForLLM`

The adapter already supports `allowlist` and `disabledTools`. May need a new option:

```typescript
interface GetToolDefinitionsOptions {
  disabledTools?: string[];
  allowlist?: string[];
  coreTools?: string[];                  // NEW: always included regardless of categories
  activeCategories?: ToolCategory[];     // NEW: dynamically activated categories
}
```

### 5. System Prompt Category Descriptions

Generate a summary of available-but-not-yet-active categories for the system prompt. This should be auto-generated from the registry so it stays in sync with actual tool definitions.

```typescript
function describeAvailableCategories(
  allCategories: ToolCategory[],
  activeCategories: ToolCategory[],
  disabledTools: string[]
): string {
  // Returns formatted string describing each inactive category
  // with tool count and one-line description
}
```

### 6. Dev UI Integration

The Tool Editor already has per-tool enable/disable. Extend it with:
- A "Core Tools" section showing which tools are always present
- A visualization of token savings (estimated tokens with all tools vs. core-only)
- Ability to set the core tool list per agent

### 7. Metrics / Observability

Track tool activation patterns to optimize defaults:
- Which categories get activated most often per agent?
- How many turns before activation happens?
- Are there tools that are activated but never called?

This data would inform better core tool defaults over time.

## Impact on Existing Agents

This should be **backward compatible**. Agents that don't set `coreTools` or use `activate_tools` continue to work exactly as they do today — all enabled tools are present on every turn.

The dynamic loading is opt-in: an agent opts in by declaring a `coreTools` list and including `activate_tools` in its available tools.

For pipeline agents with short, focused workflows (course-review, template-fidelity), static tool loading is fine. Dynamic loading primarily benefits conversational agents with broad capabilities and long conversations (the ID Assistant, and potentially future chat-based agents).

## Token Budget Comparison

| Approach | Tools per turn | ~Tokens for tools |
|----------|---------------|-------------------|
| Current (all enabled) | ~70 | 3,500-7,000 |
| Core only + category descriptions | ~8 + descriptions | ~800-1,200 |
| After activating 2 categories | ~28 | ~1,600-2,400 |
| Full search (Tier 3) | ~8 + search tool | ~600-900 |

The Tier 2 approach saves 2,000-5,000 tokens per turn on average, which adds up significantly over a multi-turn conversation.

## Design Decisions to Make

- [ ] Should `activate_tools` be a framework-level tool or agent-defined?
- [ ] Should categories auto-deactivate after N turns of non-use, or stay active until explicitly deactivated?
- [ ] Should the LLM be able to activate individual tools by name (fine-grained) or only by category (coarse)?
- [ ] For pipeline agents: should the workflow engine auto-activate categories based on the step's `tools` config?
- [ ] Should there be a maximum number of simultaneously active categories?
- [ ] At what tool count threshold does Tier 3 (full search) become worth implementing?
