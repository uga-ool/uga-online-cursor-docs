# TODO: Data Schemas

## Context

The ID Assistant's data schemas are more consequential than those of other agents because the data is shared across users, cross-cutting across courses, and long-lived (potentially spanning semesters or years). Schemas need to be versioned, carefully designed, and defined in the `shared/` package so they can be consumed by multiple agents and the backend.

## Guiding Principles

1. **Version from day one.** Every top-level data file includes a `version` field. The agent checks version on load and migrates if needed (same pattern as `migrateReviewData` in course-review).

2. **Define in `shared/`.** TypeScript interfaces go in `shared/src/types/` alongside `WhoAmIResponse`, `CourseDetails`, etc. One canonical definition, not per-consumer copies.

3. **Identity fields are stable; display fields are denormalized copies.** `userId` is identity (from Brightspace, never changes). `userName` is a convenience copy that may go stale. Decide per-schema whether to resolve names at runtime or accept staleness.

4. **Journal entries are append-only.** Adding optional fields later is safe. Changing the meaning of existing fields requires migrating every entry. Get the core fields right first.

5. **Separate human input from agent inference.** Raw `content` is immutable. Extracted/inferred metadata is stored separately and can be re-derived.

## Schemas to Define

### 1. Program Registry (`id-assistant/program.json`)

```typescript
interface ProgramRegistry {
  version: string;                    // Schema version (e.g., "1.0")
  programId: string;                  // Unique program identifier
  programName: string;                // Display name
  hubOu: string;                      // Org unit ID of the hub course
  createdAt: string;                  // ISO 8601
  lastUpdated: string;                // ISO 8601

  courses: ProgramCourse[];
  team: ProgramTeam;
}

interface ProgramCourse {
  templateOu: string;                 // Org unit ID of the template/parent course
  courseCode: string;                  // Brightspace course code (e.g., "EDUC6010")
  courseName: string;                 // Display name
  semester?: string;                  // e.g., "Fall 2026"
  status: string;                     // e.g., "planning", "in-development", "review", "complete"
  discoverOfferings: boolean;         // If true, use fetchOrgUnitChildren to find semester sections
  addedBy: string;                    // userId who registered this course
  addedAt: string;                    // ISO 8601

  sme?: TeamMember;                   // Subject matter expert
  designer?: TeamMember;              // Instructional designer
}

interface TeamMember {
  userId: string;                     // whoAmI.Identifier
  name: string;                       // Display name (denormalized)
  email?: string;
  role?: string;                      // Additional role descriptor
}

interface ProgramTeam {
  programCoordinator?: TeamMember;
  adminContacts: TeamMember[];
}
```

**Decision needed**: Store `name` denormalized, or resolve at runtime via Valence API? Denormalized is simpler and works offline, but goes stale if someone changes their Brightspace display name.

### 2. Journal Entry (`id-assistant/journal/journal_{userId}.json`)

```typescript
interface JournalFile {
  version: string;
  userId: string;                     // Owner of this journal file
  entries: JournalEntry[];
}

interface JournalEntry {
  id: string;                         // Unique entry ID (nanoid or UUID)
  timestamp: string;                  // ISO 8601
  userId: string;                     // Who wrote this
  userName: string;                   // Display name (denormalized)
  userRole: string;                   // Role from program registry (designer, sme, coordinator)

  courseOu?: string;                  // Which course this relates to (optional)
  tags: string[];                     // Lightweight categorization
  content: string;                    // The actual note (free-form text, immutable)
  source: 'user-input' | 'agent-inferred' | 'conversation';

  extracted?: {                       // Agent-derived structured data (re-derivable)
    [key: string]: unknown;
  };
}
```

**Decision needed**: Should `extracted` have a defined sub-schema (typed fields like `deadline`, `person`, `module`) or remain fully unstructured (`Record<string, unknown>`)?

### 3. Skill Manifest (`index.json` in each skill folder)

```typescript
interface SkillManifest {
  id: string;                         // Unique skill identifier (kebab-case)
  version: string;                    // Skill version
  name: string;                       // Display name
  description: string;                // Short description for catalog
  keywords: string[];                 // For search and matching
  applicability: string;              // When the LLM should use this skill

  files: SkillFile[];
}

interface SkillFile {
  path: string;                       // Relative path within the skill folder
  role: 'primary' | 'reference' | 'example';
  description: string;                // What this file contains
}
```

### 4. Skills Configuration (`data/skills.json` per agent)

```typescript
interface AgentSkillsConfig {
  enabledSkills: string[];            // IDs of enabled framework-level skills
}
```

### 5. Access Control (Optional, `{agent}/access.json`)

```typescript
interface AccessControl {
  version: string;
  allowedUsers?: string[];            // userId allowlist (if set, only these users)
  allowedRoles?: string[];            // courseRole allowlist (e.g., ["Instructor"])
  denyMessage?: string;               // Custom message for denied users
}
```

## Where to Put These

```
shared/src/types/
├── brightspace.ts          ← existing (WhoAmIResponse, CourseDetails, OrgUnitChild)
├── plugin.ts               ← existing (AgentRuntimeContext, PluginManifest, etc.)
├── llm.ts                  ← existing
├── workflow.ts             ← existing
├── program.ts              ← NEW: ProgramRegistry, ProgramCourse, TeamMember, ProgramTeam
├── journal.ts              ← NEW: JournalFile, JournalEntry
├── skills.ts               ← NEW: SkillManifest, SkillFile, AgentSkillsConfig
└── access.ts               ← NEW: AccessControl
```

Export from `shared/src/index.ts` so agents can import from `@uga-brightspace/shared`.

## Validation

Consider adding Zod schemas alongside the TypeScript interfaces (in `shared/src/schemas/`) for runtime validation on load, following the existing pattern in `shared/src/schemas/plugin.ts`.

## Migration Strategy

Each loader function should check `version` and call a migrate function if the version is older than current. Pattern:

```typescript
function migrateRegistry(data: ProgramRegistry): ProgramRegistry {
  if (data.version === '1.0') {
    // v1.0 → v1.1: add new optional field with default
    data.version = '1.1';
  }
  return data;
}
```

This is the same approach used by `migrateReviewData` and `migrateFinding` in course-review.

## Design Decisions to Make

- [ ] Should schemas use strict enums (e.g., `status: 'planning' | 'in-development' | ...`) or open strings?
- [ ] Should `TeamMember.name` be resolved at runtime or stored denormalized?
- [ ] Should `JournalEntry.extracted` be typed or unstructured?
- [ ] Should Zod schemas be required for all types, or only for types that are loaded from untrusted sources (course files)?
- [ ] Should there be a `lastModifiedBy` field on the registry for audit purposes?
