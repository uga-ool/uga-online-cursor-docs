# Market Research Agent — LLM Cost Optimization

This document records the optimizations made to reduce token usage and API costs for the market research chat agent, along with measured results across three iterations of the same benchmark prompt.

## Benchmark Prompt

A complex multi-institution analysis requesting year-by-year conferral data across multiple programs, institutions, and CIP codes — representative of the most expensive queries this agent handles.

## Results Summary

| Metric | Baseline | Iteration 2 | Iteration 3 |
|---|---|---|---|
| API calls | 7 | 7 | 5 |
| Total cost | ~$0.99 | ~$0.47 | ~$0.39 |
| Reduction | — | -52% | -60% |
| Quality | Good | Good | Good |

---

## Iteration 1 → 2: Caching, Truncation, and Streaming

### Problem

- The system prompt was not being loaded from `main.md`; a leaner fallback was used instead, causing the model to call `list_tables` and `get_table_schema` on every turn.
- Full SQL query results (up to 500 rows of raw JSON) were appended to `loopMessages` and re-sent to the LLM on every subsequent tool-call iteration, causing token costs to snowball.
- `MAX_HISTORY` was set to 20 messages, adding unnecessary context from older turns.
- LLM calls routed through API Gateway, which has a 30-second hard timeout — long responses returned 503 errors.

### Changes

**System prompt with full schema context** (`prompts/main.md`, `hooks/useChat.ts`)
- Loaded `main.md` via Vite `?raw` import and processed with `stripFrontmatter`/`interpolate`.
- Embedded complete table schemas (columns, types, descriptions, filter patterns) directly in the prompt so the model can write SQL immediately without discovery calls.

**Reduced conversation history** (`hooks/useChat.ts`)
- `MAX_HISTORY` reduced from 20 to 10 messages.

**`max_rows` parameter** (`tools/duckdbTools.ts`)
- Added an optional `max_rows` parameter to `run_sql_query` (default: 100, hard cap: 2,000).
- The model decides an appropriate limit based on query complexity.

**Client-side result caching** (`tools/duckdbTools.ts`)
- Every `run_sql_query` result is stored in a `Map` keyed by `result_id`.
- A new `get_query_result` tool lets the model retrieve specific row slices from cached results without re-executing SQL.
- Cache is cleared when the user starts a new conversation.

**History truncation** (`hooks/useChat.ts`, `tools/duckdbTools.ts`)
- Tool results from *prior* loop iterations are truncated to 20 preview rows before being sent in subsequent API calls.
- The *current* iteration's results remain full so the model can analyze them on first pass.
- `truncateToolResult()` preserves column names and row count metadata.

**Streaming via Lambda Function URLs** (`core/llm/client.ts`)
- Added `callLLMWithToolsStream` — streams tool-enabled responses directly via the Lambda Function URL, bypassing API Gateway's 30-second timeout.
- Parses SSE stream for both content deltas and tool-call deltas.
- Enables real-time text streaming in the chat UI.

**Prompt guidance** (`prompts/main.md`)
- Added instructions for efficient query planning: batch lookups, use CTEs, minimize tool calls.
- Added result caching documentation so the model uses `get_query_result` instead of re-running queries.
- Removed the instruction that told the model to call `list_tables`/`get_table_schema` on every turn.

### Impact

Cost dropped from ~$0.99 to ~$0.47 (52% reduction). Call count remained at 7 — the model was still making CIP code formatting errors that required retries.

---

## Iteration 2 → 3: CIP Code Formatting and Logging

### Problem

- CIP codes are stored as dotted varchar strings (`'52.0201'`) but the model was querying them as plain integers or unquoted strings (`520201`), causing 0-row results and retry calls.
- Tool call bubbles in the chat UI truncated SQL to 60 characters, making it impossible to spot these errors without checking LiteLLM logs.

### Changes

**CIP code guidance in prompt** (`prompts/main.md`)
- Added a prominent `### CRITICAL: CIP codes are dotted strings` section near the top of the SQL instructions.
- Provides correct (`'52.0201'`) and incorrect (`520201`, `'520201'`) examples.
- Includes self-recovery hint: "If a query returns 0 rows, check CIP code format."
- Updated all column descriptions to say "dotted format (XX.XXXX)".

**Expandable tool call bubbles** (`components/ChatView.tsx`, `hooks/useChat.ts`, `styles/market-research.css`)
- `run_sql_query` calls now show the tool name with a clickable expand/collapse toggle.
- Expanding reveals the full SQL query text, including model-generated comments.
- `max_rows` value shown as a badge when set.
- Non-SQL tools (`list_tables`, `get_table_schema`) show just the tool name.
- Removed the 60-character truncation from `formatArgs`.

**Structured tool call data** (`hooks/useChat.ts`)
- `ChatMessage` now carries `toolArgs` (the raw parsed arguments object) alongside the formatted `content` string, enabling the UI to render arguments semantically.

### Impact

Cost dropped from ~$0.47 to ~$0.39 (additional 17% reduction). Call count dropped from 7 to 5 — eliminating the wasted retry calls from CIP formatting errors. The expandable SQL bubbles confirmed all CIP codes were correctly formatted as `'XX.XXXX'`.

---

## Architecture Overview

```
User message
    │
    ▼
useChat.ts ── buildApiMessages()
    │         • System prompt from main.md (full schemas)
    │         • Last 10 history messages
    │         • Tool results from prior iterations truncated to 20 rows
    │
    ▼
callLLMWithToolsStream()  ── Lambda Function URL (no API GW timeout)
    │
    ▼
Model response
    ├─ Content chunks → streamed to UI in real time
    └─ Tool calls → dispatched to duckdbTools.ts
                        │
                        ▼
                   DuckDB WASM (in-browser SQL)
                        │
                        ▼
                   Result cached by result_id
                   Full result on first pass; truncated in history
                        │
                        ▼
                   Loop continues until no more tool calls
```

## Key Files

| File | Role |
|---|---|
| `prompts/main.md` | System prompt with table schemas, query planning guidance, CIP format rules |
| `hooks/useChat.ts` | Chat loop, history management, truncation logic |
| `tools/duckdbTools.ts` | Tool definitions, DuckDB dispatch, result caching, `truncateToolResult` |
| `components/ChatView.tsx` | Chat UI with expandable tool call bubbles |
| `styles/market-research.css` | Tool call bubble styling |
| `core/llm/client.ts` | `callLLMWithToolsStream` for streaming with tool calls |
