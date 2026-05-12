# TODO: Identity & Access Control

## Context

The framework establishes user identity by inheriting the Brightspace session — no separate auth system. On startup, `BrightspaceProvider` calls the Valence `/users/whoami` endpoint to get `Identifier`, `FirstName`, `LastName`, and `ProfileIdentifier`, and fetches the user's enrollment to get `courseRole`. The `whoAmI.Identifier` is the canonical user ID throughout the system (LLM metadata, OAuth token storage, logging).

For the ID Assistant agent (and potentially others), we need to layer **access control** and **role-aware behavior** on top of this existing identity.

## What Exists Today

- `BrightspaceProvider` / `useBrightspace()` — provides `whoAmI`, `courseRole`, `xsrfToken`, `ou`
- `whoAmI.Identifier` propagated to backend via `setLLMRequestMetadata()` for logging/tracing
- `validateSession` middleware on backend (optional strict enforcement via `REQUIRE_BRIGHTSPACE_SESSION`)
- Brightspace enrollment API gives `courseRole` (e.g., Instructor, Student)
- OAuth token storage keyed by `userId` + `service` in DynamoDB

## Work Items

### 1. Role-Based Access Control (Agent-Level)

An agent should be able to restrict access based on `courseRole`. This is a simple check on mount — if the user's role doesn't match, render an access-denied message instead of the agent UI.

- **Approach**: Check `courseRole` from `useBrightspace()` against an allowed-roles list
- **Where**: Agent-level logic (not a framework gate), but could become a reusable `<RequireRole>` wrapper
- **Example**: `if (courseRole !== 'Instructor') return <AccessDenied />`

### 2. User-Level Access Control (Allowlist)

For fine-grained control (e.g., "only these 5 people can use this agent"), use a JSON allowlist file stored in course files.

- **File**: `{agent}/access.json` in the course file system
- **Format**: `{ "allowedUsers": ["12345", "67890"], "allowedRoles": ["Instructor"] }`
- **Loading**: Via existing `lms-file` source type in `InputDeclaration`
- **Check**: Match `whoAmI.Identifier` against the allowlist on agent startup
- **Fallback**: If no `access.json` exists, default to open access (or role-based only)

### 3. Registry-Based Access (ID Assistant Specific)

For the ID Assistant, the program registry (`id-assistant/program.json`) doubles as the access control list. If a user's `Identifier` appears in the team roster, they have access. Their role in the registry (SME, designer, coordinator) determines how the agent behaves.

- **See**: `TODO-DATA-REGISTRY.md` for the registry schema
- **Match**: `whoAmI.Identifier` against `team` and `courses[].sme.userId`, `courses[].designer.userId`, etc.
- **Behavior**: System prompt changes based on the user's project role (not just their Brightspace role)

### 4. Identity Propagation for Cross-Course Access

When the agent reads data from courses other than the hub (see `TODO-DATA-LOADING.md`), the Brightspace session handles authorization implicitly — `withCredentials: true` sends the user's cookies, and the Valence API checks their permissions on the target org unit.

- **No framework changes needed** for this
- **Agent should handle gracefully**: If `readCourseFile` returns `null` for a registered course, surface a message like "I don't have access to EDUC 6020's files — you may need to be enrolled in that course"

## Design Decisions to Make

- [ ] Should `<RequireRole>` be a framework-level component or stay agent-level?
- [ ] Should access denial be a hard block (no UI at all) or a soft block (UI visible but functionality disabled)?
- [ ] For the registry-as-access-control pattern: what happens when a user is enrolled in the hub course but not in the registry? Deny access entirely, or allow read-only/limited access?
