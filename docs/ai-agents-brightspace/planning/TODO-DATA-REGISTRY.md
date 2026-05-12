# TODO: Program Registry & The Program Hub Model

## Context

The ID Assistant agent lives in a single "Program Hub" course and needs to access data from multiple development course areas. The program registry is a JSON file in the hub course's file system that serves as the single source of truth for which courses belong to the program, who the team members are, and their roles.

The registry is **runtime data, not build data**. The agent ships with no program-specific data. On first launch it presents an empty state and walks the user through setup. The same built agent can be deployed into multiple hub courses, each with its own independent registry.

## What Exists Today

- `brightspaceClient.readCourseFile(ou, path)` — reads JSON from any org unit (already parameterized by `ou`)
- `brightspaceClient.fetchCourseDetails(ou)` — validates an org unit exists, returns name/code
- `brightspaceClient.fetchOrgUnitChildren(orgUnitId, ouTypeId)` — discovers child offerings (used by template-fidelity)
- `parseSemesterFromOrgUnitCode(code)` — parses semester from Brightspace course codes
- `createCourseFolder` / `uploadFileToCourse` — writes JSON files to course file system
- `InputDeclaration` with `sourceType: 'lms-file'` — declarative data loading from course files

## Work Items

### 1. Define the Program Registry Schema

Store at `id-assistant/program.json` in the hub course files. See `TODO-DATA-SCHEMAS.md` for detailed schema design.

Key fields:
- **Program metadata**: `programId`, `programName`, `hubOu`
- **Courses**: Array of registered courses, each with `templateOu`, `courseCode`, `courseName`, `addedBy`, `addedAt`, `discoverOfferings` flag
- **Team**: `programCoordinator`, `adminContacts`, plus per-course `sme` and `designer` on each course entry
- **Version**: Schema version for future migrations

### 2. Registry Management UI

An admin panel in the agent for managing the registry. Not raw JSON editing.

- **Add Course**: User pastes a Brightspace course URL or enters an org unit ID. Agent calls `fetchCourseDetails(ou)` to validate and pull name/code. Adds to registry.
- **Remove Course**: Remove a course from the registry (does not affect the actual course)
- **Assign Team Members**: For each course, assign SME, designer. For the program, assign coordinator and admin contacts. Consider pulling from course classlists via `get_classlist` to let the user pick from enrolled users.
- **Save**: Write updated registry back to course files via `uploadFileToCourse`

### 3. Offering Discovery

When `discoverOfferings: true` on a registered course, use `fetchOrgUnitChildren(templateOu, 3)` at runtime to find all semester sections under that template. This follows the pattern already used by template-fidelity's `offeringsService.ts`.

- Register the template course once; agent discovers active semester offerings dynamically
- Use `parseSemesterFromOrgUnitCode` to label offerings by semester
- Do NOT let the LLM autonomously discover and add courses — discovery should be scoped to children of registered templates

### 4. Cross-Course Data Access

The agent reads data from registered courses using the same `brightspaceClient` methods, just passing different `ou` values.

- `readCourseFile(targetOu, path)` for reading agent state/output from other courses
- User's Brightspace session handles authorization implicitly
- Handle `null` returns gracefully (user may not have permissions on a registered course)

### 5. Expose `ou` as LLM Tool Parameter

The tool executor's `resolveUrl` already supports `args.ou` overriding `ctx.ou`, but the tool definitions in `course.ts` don't declare `ou` as a parameter. The LLM doesn't know it can pass one.

Options:
- Add an optional `ou` parameter to existing tools (`get_manage_files`, `read_course_file`, `get_course_details`, etc.)
- Or create a small set of cross-course-specific tools (e.g., `read_project_course_file`) that take `ou` explicitly
- The agent's system prompt should list registered course OUs so the LLM knows which values to use

## Design Decisions to Make

- [ ] Should the registry support multiple programs per hub course, or one-to-one?
- [ ] Should team member names be stored in the registry (denormalized) or resolved at runtime from the Valence API?
- [ ] Should the registry store Brightspace URLs alongside org unit IDs for human readability?
- [ ] How should the agent handle a registered course that the current user doesn't have access to?
