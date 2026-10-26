# API Reference

## Overview

Quality Scorecard for Jira has no public HTTP API. All frontend ↔ backend communication happens through **Forge resolver functions** defined in `src/index.js` and called from the Custom UI with `invoke()` from `@forge/bridge`. This document lists every resolver, its payload, its response shape and the authorization rule the backend enforces.

For security implementation details, see [Security Documentation](./Security.md).
For development setup, see [Development Guide](./Development.md).
For troubleshooting, see [Technical Troubleshooting](./Troubleshooting-Technical.md).

## Calling Conventions

```typescript
import { invoke } from '@forge/bridge';

const result = await invoke('getMyPermissions', { projectKey: 'PROJ' });
```

In the app, calls go through `src/services/utils/api.ts`, which adds:
- a global concurrency limiter (max 8 in-flight calls),
- in-flight de-duplication of identical calls,
- `withRetry()` – up to 3 attempts with linear back-off.

### Authentication context
Forge authenticates every call and provides `context.accountId` to the resolver. Resolvers use:
- `api.asUser()` – Jira calls made with the caller's permissions (issue search, project search, permission checks, `serverInfo`).
- `api.asApp()` – Jira calls made with the app's permissions (user look-ups, batch ticket metadata after a BROWSE_PROJECTS check, project avatars).

### Error semantics
Most resolvers **throw** on authorization or validation failures (the `invoke()` promise rejects with an `Error` whose `message` describes the problem). Several analytics/settings resolvers instead return `{ success: false, error: string }`. Each entry below states which pattern applies. There is no numeric error-code scheme.

### Permission vocabulary
| Permission | Grants |
|------------|--------|
| `dashboard.access` | Open the project dashboard |
| `tab.analytics` | Analytics Dashboard tab |
| `tab.agentPerformance` | Agent Performance tab |
| `tab.evaluatorAnalytics` | Evaluator Analytics tab |
| `tab.advancedAnalytics` | Advanced Analytics tab |
| `tab.configuration` | Metrics tab (criteria editor) |
| `tab.settings` | Settings tab and role audit log |
| `ticket.evaluate` | Save evaluations |
| `evaluation.delete` | Delete all evaluations of a ticket |

Jira administrators (global `ADMINISTER` permission, or members of the group named in the `settings-admin-group` storage key) implicitly hold every permission. "Admin" below means this check (`checkAdminPermission`).

---

## Jira Data

### getCurrentUser
Returns the calling user.

```typescript
invoke('getCurrentUser')
// → { id: string, displayName: string, avatarUrl: string | null }
```
Display name resolution order: manual mapping (`evaluator-name-mapping`) → Forge context → `/rest/api/3/myself`. Never returns the email address.

### getIssue
```typescript
invoke('getIssue', { issueId: string })   // numeric id or KEY-123
// → Jira issue JSON (fields.summary, status, priority, issuetype, assignee, ...)
```
Validation: `issueId` must match `^([A-Z][A-Z0-9_-]{1,9}-)?[0-9]{1,10}$`. Throws on failure.

### searchIssues
```typescript
invoke('searchIssues', {
  jql: string,
  fields?: string,          // comma-separated; default 'id,key,summary,assignee,reporter,status,issuetype,priority,created,updated'
  maxResults?: number,      // 1–100, default 100
  startAt?: number,         // 0–100000
  nextPageToken?: string    // token pagination (new Jira search API)
})
// → { issues: JiraIssue[], total: number, nextPageToken?: string, startAt: number, maxResults: number }
```
Behaviour:
- `jql` is validated (≤ 5000 chars; `;`, `--`, `/*`, `\\` and control characters are rejected).
- The query is wrapped as `project in ("KEY1", "KEY2") AND (<jql>)` using the **allowed projects** list. If no projects are allowed, `{ issues: [], total: 0 }` is returned without calling Jira.
- Calls `POST /rest/api/3/search`; on HTTP 410 it transparently falls back to `POST /rest/api/3/search/jql` (token-based pagination).
- Executed `asUser`, so Jira's own permissions apply.

### getTicketMetadataBatch
```typescript
invoke('getTicketMetadataBatch', { ticketIds: string[], projectKey: string })
// → { success: true, tickets: TicketMetadata[] } | { success: false, error }
// TicketMetadata = { id, key, summary, status, priority, created, updated, assignee: {accountId, displayName} | null, issueType }
```
Authorization: `projectKey` must be in the allow-list and the caller must hold Jira `BROWSE_PROJECTS` on it (checked `asUser`); the lookup itself runs `asApp` in batches of 50 numeric ids, scoped with `AND project = "<KEY>"`.

### getJiraBaseUrl
```typescript
invoke('getJiraBaseUrl')   // → { baseUrl: string | null }
```
Used to build "open in Jira" links.

### searchProjects
```typescript
invoke('searchProjects', { query?: string })
// → { values: { id, key, name, projectCategory, avatarUrl }[] }   (max 50)
```
Runs `asUser`; used by Settings → Project Access.

### searchUsers
```typescript
invoke('searchUsers', { query?: string, projectKey?: string, startAt?: number, maxResults?: number /* default 50 */ })
// → { success: true, users: { accountId, displayName }[] }
```
With `projectKey` it searches assignable users for that project (24 h server-side cache in `assignable-users-cache:<KEY>`); otherwise it uses the global user search. Falls back to the manual evaluator-name mapping if Jira returns nothing.

### getUser
```typescript
invoke('getUser', { accountId: string })
// → { accountId, displayName, avatarUrl? } | null
```

### batchGetUsers
```typescript
invoke('batchGetUsers', { accountIds: string[] })
// → { [accountId]: { accountId, displayName, ... } }
```
Resolves names via server-side caches (`user-cache:<id>`), manual mapping and `GET /rest/api/3/user`.

---

## Storage (allow-listed keys)

The three generic storage resolvers only accept the keys listed here. Any other key is rejected with `Storage key not permitted for <read|write|delete>` and an `ACCESS_DENIED` audit event is written.

### getStorageValue
```typescript
invoke('getStorageValue', { key: string })   // → stored value | undefined
```
Readable keys: `ticket-evaluations`, `project-evaluations-<KEY>` (project must be allowed), `evaluation-criteria`, `coverage-targets`, `theme-preference`, `selected-project`, `user-displayname-cache`, `evaluator-name-mapping`, `removed-missing-evaluations`.

### setStorageValue
```typescript
invoke('setStorageValue', { key: string, value: any })   // → { success: true }
```
Writable keys and rules:

| Key | Rule |
|-----|------|
| `ticket-evaluations`, `project-evaluations-<KEY>` | Caller must hold `ticket.evaluate` in the project derived from the key or from the last element's `ticketKey`. Clearing the global list (no derivable project) requires admin. Account ids in the saved evaluation are recorded in the personal-data index. |
| `evaluation-criteria` | Anyone may seed it when empty (first run); once it exists, editing requires admin. |
| `theme-preference`, `selected-project`, `user-displayname-cache`, `removed-missing-evaluations` | Any authenticated user. |

### deleteStorageValue
```typescript
invoke('deleteStorageValue', { key: 'selected-project' })   // → { success: true }
```
`selected-project` is the only deletable key.

---

## Evaluations

Evaluations are written through `setStorageValue` (see above) by `evaluationService.ts`, which appends the new record to both `ticket-evaluations` and `project-evaluations-<KEY>`. Reading uses `getStorageValue` or the resolver below.

### getFilteredEvaluations
```typescript
invoke('getFilteredEvaluations', {
  projectKey?: string,
  filters?: { evaluatorId?: string, minScore?: number, maxScore?: number }
})
// → { success: true, evaluations: TicketEvaluation[] } | { success: false, error }
```
Merges the global and project-scoped lists, keeps records whose `ticketKey` starts with `projectKey`, then applies the evaluator and rounded-score filters. Date, assignee, priority and ticket-type filters are applied **client-side** after ticket metadata is fetched.

### deleteTicketEvaluations
```typescript
invoke('deleteTicketEvaluations', { projectKey: string, ticketKey: string })
// → { success: true, deleted: number }
```
Authorization: `evaluation.delete` in the project. Removes every evaluation for the ticket from both storage keys, writes a `PERMISSION_CHANGED` audit event (`action: 'delete_evaluations'`) and a role-audit entry. Throws when unauthorized.

---

## Evaluator Display Names

### setEvaluatorName (admin)
```typescript
invoke('setEvaluatorName', { evaluatorId: string, displayName: string })   // → { success: true }
```
### removeEvaluatorName
```typescript
invoke('removeEvaluatorName', { evaluatorId: string })   // → { success: true }
```
Removes the manual override and records the id in `evaluator-name-removed`.
### getEvaluatorNameMapping
```typescript
invoke('getEvaluatorNameMapping')   // → { success: true, mapping: Record<accountId, displayName> }
```
### migrateEvaluatorNames
```typescript
invoke('migrateEvaluatorNames')   // → { success: true, migrated: number, ... }
```
Back-fills display names for evaluator ids found in stored evaluations.
### clearEvaluatorNameCache (admin)
```typescript
invoke('clearEvaluatorNameCache')   // → { success: true }
```
> The Settings UI that used these resolvers (`EvaluatorNameManager`, `SmartEvaluatorSync`) is not mounted in the current build; names are resolved automatically via `batchGetUsers`/`searchUsers`.

---

## Project Access

### getAllowedProjects
```typescript
invoke('getAllowedProjects')
// → { projects: { key, name, avatarUrl, addedBy, addedAt }[], audit: AuditEntry[] }   // audit = last 20, newest first
```
### setAllowedProjects (admin)
```typescript
invoke('setAllowedProjects', { action: 'add' | 'remove', projects: { key: string, name: string, avatarUrl?: string }[] })
// → { success: true }
```
Each change is appended to `allowed-projects-audit` as `{ action: 'added'|'removed', projectKey, projectName, byAccountId, byDisplayName, atIso }`.

### checkAdminPermission
```typescript
invoke('checkAdminPermission')   // → boolean
```

---

## Coverage Targets

### getCoverageTargets
```typescript
invoke('getCoverageTargets')
// → { success: true, targets: { overallTarget: number, byPriorityTargets: { Highest, High, Medium, Low, Lowest } } }
// defaults: overallTarget 75; Highest 100, High 100, Medium 50, Low 30, Lowest 20
```
### setCoverageTargets (admin)
```typescript
invoke('setCoverageTargets', { targets: { overallTarget: number, byPriorityTargets: Record<string, number> } })
// → { success: true } | { success: false, error }
```

---

## Roles & Permissions (RBAC)

All RBAC resolvers require `projectKey` to be in the allow-list (`ensureProjectAccessConfigured`). Storage: `roles:<KEY>`, `role-assignments:<KEY>`, `role-audit:<KEY>`.

### getPermissionCatalogue
```typescript
invoke('getPermissionCatalogue')   // → { success: true, permissions: PermissionKey[] }
```
### getProjectRoles
```typescript
invoke('getProjectRoles', { projectKey })   // → { success: true, roles: { [roleName]: { name, permissions } } }
```
### upsertProjectRole (admin)
```typescript
invoke('upsertProjectRole', { projectKey, name: string, permissions: PermissionKey[] })
// → { success: true, role: { name, permissions } }
```
Unknown permission strings are silently dropped.
### deleteProjectRole (admin)
```typescript
invoke('deleteProjectRole', { projectKey, name })   // → { success: true }
```
Also removes the role from every user's assignment list.
### assignUserRoles (admin)
```typescript
invoke('assignUserRoles', { projectKey, accountId, roleNames: string[] })   // → { success: true }
```
Replaces the user's roles in the project and records the account in the personal-data index.
### bulkAssignUserRoles (admin)
```typescript
invoke('bulkAssignUserRoles', { projectKey, assignments: Record<accountId, string[]> })   // → { success: true }
```
### removeUserAssignment (admin)
```typescript
invoke('removeUserAssignment', { projectKey, accountId })   // → { success: true }
```
### getUserAssignments (`tab.settings`)
```typescript
invoke('getUserAssignments', { projectKey })   // → { success: true, assignments: Record<accountId, string[]> }
```
### getMyPermissions
```typescript
invoke('getMyPermissions', { projectKey })   // → { success: true, permissions: PermissionKey[] }
```
### canAccess
```typescript
invoke('canAccess', { projectKey, resource?: PermissionKey })   // → { success: true, allowed: boolean }
```
Returns `false` when the caller lacks `dashboard.access`; otherwise checks `resource` (or returns `true` if omitted).
### getRoleAuditLog (`tab.settings`)
```typescript
invoke('getRoleAuditLog', { projectKey, limit?: number /*20*/, offset?: number /*0*/, startDate?: string, endDate?: string })
// → { success: true, logs: { ...rawEntry, actor: string, action: string, description: string }[] } | { success: false, error }
```
Raw entries come from `role-audit:<KEY>` (`role_upsert`, `role_delete`, `assign_roles`, `bulk_assign_roles`, `remove_user`, `delete_evaluations`). The resolver resolves account ids to display names and adds friendly labels such as "Role Creation", "User Added", "User Removed".

---

## Data Models

```typescript
interface TicketEvaluation {
  id: string;
  ticketId: string;
  ticketKey: string;
  evaluatorId: string;
  evaluatorDisplayName?: string;
  timestamp: string;                 // ISO 8601
  criteria: Record<string, number>;  // criterionId → 0..10
  averageScore: number;
  feedback?: string;
  version?: number;
}

interface EvaluationCriteria {
  id: string;
  name: string;
  description: string;
  required: boolean;
  defaultValue: number;              // 1..10
}
// 'evaluation-criteria' stores EvaluationCriteria[] (ordered)

interface CoverageTargets {
  overallTarget: number;
  byPriorityTargets: Record<'Highest'|'High'|'Medium'|'Low'|'Lowest', number>;
}

type PermissionKey =
  | 'dashboard.access' | 'tab.analytics' | 'tab.agentPerformance' | 'tab.evaluatorAnalytics'
  | 'tab.advancedAnalytics' | 'tab.configuration' | 'tab.settings' | 'ticket.evaluate' | 'evaluation.delete';

interface RoleDefinition { name: string; permissions: PermissionKey[]; }
```

---

## Input Validation Reference (`src/validators/inputValidator.js`)

| Validator | Rule |
|-----------|------|
| `validateJQL` | non-empty string, ≤ 5000 chars, rejects `;`, `--`, `/*`, `\\`, control characters |
| `validateProjectKey` | `^[A-Z][A-Z0-9_-]{1,9}$` |
| `validateIssueId` | `^([A-Z][A-Z0-9_-]{1,9}-)?[0-9]{1,10}$` |
| `validateAccountId` | `^[a-zA-Z0-9:_-]{1,255}$` |
| `validateStorageKey` | `^[a-zA-Z0-9:_-]{1,200}$` |
| `validatePagination` | `startAt` 0–100000, `maxResults` 1–100 |
| `sanitizeString` | strips control characters, trims, truncates to 10 000 chars |

---

## Rate Limiting and Pagination

- There is **no application-level rate limiter**. Burst control is client-side (8 concurrent calls, de-duplication). Atlassian's Forge invocation limits and Jira REST limits apply; a 429 from Jira surfaces as a thrown error after the retry attempts are exhausted.
- Jira search pagination: use `startAt`/`maxResults` on the classic endpoint, or pass back `nextPageToken` when it is returned.

```typescript
let token: string | undefined;
do {
  const page = await invoke('searchIssues', { jql, maxResults: 100, nextPageToken: token });
  all.push(...page.issues);
  token = page.nextPageToken;
} while (token);
```

## Removed Resolvers

The following resolvers existed in earlier builds and were removed during the security hardening before Marketplace submission: `makeApiRequest` (generic Jira proxy), `debugUserFetch`, `testUserAPI`, `runPersonalDataReporter`. Any integration relying on them must use the purpose-built resolvers above.

## Related Documentation

- [Architecture Overview](./Architecture.md)
- [Installation Guide](./Installation.md)
- [Development Guide](./Development.md)
- [Security Documentation](./Security.md)
- [Technical Troubleshooting](./Troubleshooting-Technical.md)

---

*Last Updated: October 2026 — reflects `src/index.js` in app version 1.3.0*
