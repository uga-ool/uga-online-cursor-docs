# TODO: Cross-Course Data Loading

## Context

The ID Assistant runs in a Program Hub course but needs to read data from multiple development courses. The framework's Brightspace API layer is already parameterized by `ou`, so cross-course reads work mechanically. The main work is in making the LLM aware of cross-course capabilities and handling permissions gracefully.

## What Exists Today

- All `brightspaceClient` methods take `ou` as an explicit argument — not a global
- `resolveUrl` in `tools/executor.ts` already supports `args.ou` overriding `ctx.ou` (line 35: `const ou = args.ou != null ? String(args.ou) : ctx.ou`)
- All requests use `withCredentials: true`, so the user's Brightspace session credentials travel with cross-course API calls
- `readCourseFile` returns `null` on error (silent failure) — agent needs to distinguish "file doesn't exist" from "no permissions"
- `fetchOrgUnitChildren` discovers child org units (used by template-fidelity for offerings)
- `loadInput` / `dataLoader.ts` is scoped to `context.ou` — only loads from the current course

## Work Items

### 1. Expose `ou` on LLM Tool Definitions

The executor already handles `args.ou`, but the tool definitions in `core/tools/builtins/course.ts` don't declare `ou` as a parameter. The LLM can't pass what it doesn't know exists.

**Option A** — Add optional `ou` parameter to existing tools:
```typescript
{ 
  name: 'read_course_file', 
  // ... existing definition ...
  params: [
    { name: 'ou', type: 'string', description: 'Org unit ID (defaults to current course)', required: false },
    { name: 'path', type: 'string', description: 'Path to file', required: true, isQueryParam: true }
  ]
}
```

**Option B** — Create cross-course wrapper tools:
```typescript
{ 
  name: 'read_project_course_file', 
  description: 'Read a file from a registered project course.',
  params: [
    { name: 'ou', type: 'string', description: 'Target course org unit ID', required: true },
    { name: 'path', type: 'string', description: 'File path', required: true }
  ]
}
```

**Recommendation**: Option A is simpler and doesn't create parallel tool sets. The system prompt can list the registered course OUs so the LLM knows which values to use.

### 2. System Prompt with Course Context

When the agent builds its system prompt, include the registered courses from the program registry:

```
You have access to the following project courses:
- EDUC 6010 (OU: 12345) — Foundations of Education — SME: Dr. Jones — Status: in-development
- EDUC 6020 (OU: 67890) — Curriculum Design — SME: Dr. Garcia — Status: planning

Use the ou parameter on tools like read_course_file and get_manage_files to access these courses.
```

### 3. Permission-Aware Error Handling

Currently `readCourseFile` catches all errors and returns `null`. For cross-course access, the agent should distinguish:
- File doesn't exist → expected, handle gracefully
- 403/401 → user doesn't have permissions on that course → surface a meaningful message
- Network error → transient, maybe retry

Consider whether `readCourseFile` (or a new variant) should surface error details for the cross-course case rather than swallowing them.

### 4. Reading Other Agents' Output

The ID Assistant should be able to read state/output from other agents running in the development courses:
- Course review results: `course-review/review_{standardSetId}.json`
- Module builder output: `module_builder/*.md`
- Embeddings manifest: `embeddings/manifest.json`
- Template fidelity reports: `template-fidelity/reports.json`

This requires no new APIs — just `readCourseFile(targetOu, path)` with the known file paths. The agent's system prompt or skills should document what data is available and where.

### 5. DataLoader Extension (Optional)

The current `loadInput` / `loadLmsFile` always uses `context.ou`. For the hub model, it might be useful to have a `loadInputFromOu(declaration, targetOu, context)` variant that reads from a specific org unit.

This is a convenience — you can always call `brightspaceClient.readCourseFile` directly. But if the `InputDeclaration` pattern is used for cross-course data, the loader should support it.

## Design Decisions to Make

- [ ] Option A vs Option B for cross-course tool access?
- [ ] Should `readCourseFile` surface error details (status code) or continue returning `null`?
- [ ] Should the agent pre-fetch summaries of other agents' data from registered courses on startup, or only fetch on demand?
- [ ] Should cross-course writes be supported (e.g., the ID Assistant writing notes to a development course), or keep it read-only?
