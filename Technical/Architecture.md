# Technical Architecture

## Overview

Quality Scorecard for Jira is an Atlassian Forge app that adds a ticket-quality evaluation workflow and analytics dashboards to Jira Cloud. This document describes the actual architecture of the shipped app: Forge modules, the React frontend, the resolver backend, storage layout, caching, authorization and the scheduled Personal Data Reporting job.

**Quick Links:**
- [Installation Guide](./Installation.md) - Deployment and first-run configuration
- [API Reference](./API-Reference.md) - Resolver functions the frontend can invoke
- [Security Documentation](./Security.md) - Authorization model, validation, audit logging, GDPR
- [Development Guide](./Development.md) - Local development workflow

## System Architecture

### High-Level Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                           Jira Cloud                             │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                  Forge Platform Runtime                    │  │
│  │                                                            │  │
│  │   jira:globalPage ──┐      jira:issuePanel ──┐             │  │
│  │   (dashboard)       │      (evaluation form) │             │  │
│  │          ┌──────────▼────────────────────────▼──────────┐  │  │
│  │          │        React frontend (Custom UI, dist/)     │  │  │
│  │          │  @forge/bridge invoke() ──► resolver function│  │  │
│  │          └──────────────────────┬───────────────────────┘  │  │
│  │                                 │                          │  │
│  │          ┌──────────────────────▼───────────────────────┐  │  │
│  │          │   Forge functions (Node.js 22)               │  │  │
│  │          │   • resolver            (src/index.js)       │  │  │
│  │          │   • personalDataReporter (scheduledTrigger)  │  │  │
│  │          └──────────┬───────────────────────┬───────────┘  │  │
│  │                     │                       │              │  │
│  │          ┌──────────▼──────────┐  ┌─────────▼──────────┐   │  │
│  │          │  Forge KV Storage   │  │  Jira REST API v3  │   │  │
│  │          │  (storage:app)      │  │  asUser / asApp    │   │  │
│  │          └─────────────────────┘  └────────────────────┘   │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

### Forge Modules (manifest.yml)

| Module | Key | Purpose |
|--------|-----|---------|
| `jira:globalPage` | `ticket-evaluation-dashboard` | The app's main page (Jira **Apps** menu). Hosts the project landing page and the admin dashboard. |
| `jira:issuePanel` | `ticket-evaluation-panel` | Issue-view panel that hosts the evaluation form and evaluation history. |
| `scheduledTrigger` | `personal-data-reporting-weekly` | Weekly job (`interval: week`) that runs the Personal Data Reporting handler. |
| `function` | `resolver` → `index.handler` | All `invoke()` resolvers (`src/index.js`). |
| `function` | `personalDataReporter` → `personalDataReporting.handler` | Personal Data Reporting job (`src/personalDataReporting.js`). |

Both UI modules use Custom UI resources that point at the Vite build output (`resources[].path: dist`).

Runtime and permissions:

```yaml
app:
  runtime:
    name: nodejs22.x
  licensing:
    enabled: true

permissions:
  scopes:
    - read:jira-work
    - storage:app
    - read:jira-user
    - report:personal-data
```

The app does **not** request `write:jira-work`; it never modifies Jira issues. Evaluations live only in Forge storage.

## Technology Stack

### Frontend
- **Framework**: React 18 with TypeScript, bundled by Vite 6 (`@vitejs/plugin-react-swc`)
- **UI**: shadcn/ui on Radix UI primitives, Tailwind CSS, lucide-react icons
- **Routing / state**: `react-router-dom`, React Context (`ProjectContext`, `ThemeContext`, `JiraContext`)
- **Charts**: Recharts
- **Dates**: date-fns, react-day-picker
- **Exports**: ExcelJS (multi-worksheet `.xlsx`) and hand-built CSV
- **Workers**: a Web Worker (`src/services/workers/advancedAnalyticsWorker.ts`) computes advanced analytics off the main thread
- **Caching**: in-memory TTL cache (`cacheService.ts`) plus an IndexedDB persisted cache (`persistedCache.ts`, database `qc-persisted-cache`) used in a stale-while-revalidate pattern
- **Forge bridge**: `@forge/bridge` `invoke()` wrapped by `services/utils/api.ts`

### Backend (Forge functions)
- **Runtime**: Forge, Node.js 22 (`nodejs22.x`)
- **Language**: JavaScript (CommonJS)
- **Libraries**: `@forge/resolver`, `@forge/api` (`api.asUser()`, `api.asApp()`, `route`, `storage`, `privacy.reportPersonalData`)
- **Storage**: Forge Key-Value storage (`storage:app`)

## Core Components

### 1. Frontend

#### Pages (`src/pages`)
| Page | Route / host | Description |
|------|--------------|-------------|
| `Index.tsx` | global page entry | Decides which view to render based on the Forge module context. |
| `ProjectLandingPage.tsx` | global page | Lists the projects the user may open (allow-list from Settings); auto-selects when only one project is allowed. |
| `AdminView.tsx` | global page | Tabbed dashboard: **Analytics Dashboard**, **Agent Performance**, **Evaluator Analytics**, **Advanced Analytics**, **Metrics**, **Settings**. Tabs are shown only if the user holds the matching `tab.*` permission. |
| `IssuePanelView.tsx` | issue panel | Hosts `EvaluationForm` and `EvaluationHistory` for the current issue. |
| `NotFound.tsx` | fallback | 404 page. |

#### Feature components (`src/components`)
- `evaluation/EvaluationForm.tsx` – per-criterion sliders (0–10, step 1, default 5), optional feedback, **Save Evaluation** and **View History**.
- `evaluation/EvaluationHistory.tsx` – dialog listing previous evaluations for the issue with per-criterion detail.
- `admin/AnalyticsDashboard.tsx` – date-range filtered KPIs, charts and a paginated raw-data table; CSV/Excel export.
- `admin/AgentPerformanceDashboard.tsx` (+ `agent-performance/*`) – per-assignee ("agent") scores, cards, detail modal with charts and evaluated tickets.
- `admin/EvaluatorAnalyticsDashboard.tsx` (+ `evaluator-analytics/*`) – per-evaluator scoring, bias detection, evaluator detail sheet (with evaluation deletion for users holding `evaluation.delete`), and the **Missing Evaluations** table.
- `admin/AdvancedAnalyticsDashboard.tsx` (+ `advanced-analytics/*`) – coverage, trends & capacity, complexity and distribution analysis.
- `admin/CriteriaManager.tsx` – **Metrics** tab; add/edit/reorder/remove evaluation criteria.
- `admin/settings/ProjectAccessCard.tsx` – project allow-list and its "Recent Changes" audit list.
- `admin/settings/RolesAccessManager.tsx` – project roles, permission assignment, user assignment, role audit log.
- `admin/settings/CoverageTargetsCard.tsx` – overall and per-priority coverage targets.
- `shared/TicketDetailSheet.tsx` – drill-down sheet opened from charts/tables; links to the Jira issue.

> `admin/settings/EvaluatorNameManager.tsx`, `admin/settings/SmartEvaluatorSync.tsx` and `admin/analytics/{OverviewTab,AssigneeTab,CriteriaTab,DataTab}.tsx` exist in the source tree but are **not mounted** by any page. Treat them as legacy code.

#### Services (`src/services`)
```
services/
├── evaluationService.ts        # Save/load evaluations (global + project-scoped keys), history
├── criteriaService.ts          # Load/save evaluation criteria (seeds defaults on first run)
├── ticketService.ts            # Jira issue lookups via searchIssues / getTicketMetadataBatch
├── analyticsService.ts         # Analytics, agent performance, evaluator analytics, missing evaluations
├── advancedAnalyticsService.ts # Coverage / trends / complexity / distribution (uses worker + persisted cache)
├── exportService.ts            # CSV and Excel (ExcelJS) export, CSV-injection sanitisation
├── accessControlService.ts     # Thin wrapper over the roles/permissions resolvers
├── userNameService.ts          # Display-name resolution (manual mapping > Jira lookup > cache)
├── cacheService.ts             # In-memory TTL cache (SHORT 1m, MEDIUM 5m, LONG 30m, VERY_LONG 2h)
├── auditService.js             # Backend audit logging (used by src/index.js)
├── gdprService.js              # Backend erasure helper used by Personal Data Reporting
├── personalDataIndex.js        # Backend index of account IDs the app has stored
├── workers/advancedAnalyticsWorker.ts
└── utils/
    ├── api.ts                  # invoke() wrapper: concurrency limiter (8), in-flight dedupe, retry
    ├── forge-bridge.ts         # Bridge access helper
    ├── persistedCache.ts       # IndexedDB cache
    ├── storageUtils.ts         # Storage key helpers
    ├── jiraUrl.ts              # Build/open Jira issue links (uses getJiraBaseUrl)
    ├── mockData.ts             # DEFAULT_CRITERIA and dev mock data
    └── logger.ts
```

### 2. Backend Resolvers (`src/index.js`)

All frontend calls go through `invoke('<resolverName>', payload)`. The resolver file defines 35 functions grouped as:

- **Jira data**: `getCurrentUser`, `getIssue`, `searchIssues`, `getTicketMetadataBatch`, `getJiraBaseUrl`, `searchProjects`, `searchUsers`, `getUser`, `batchGetUsers`
- **Storage (allow-listed keys)**: `getStorageValue`, `setStorageValue`, `deleteStorageValue`
- **Evaluations**: `getFilteredEvaluations`, `deleteTicketEvaluations`
- **Evaluator names**: `setEvaluatorName`, `removeEvaluatorName`, `clearEvaluatorNameCache`, `migrateEvaluatorNames`, `getEvaluatorNameMapping`
- **Project access**: `getAllowedProjects`, `setAllowedProjects`, `checkAdminPermission`
- **Coverage targets**: `getCoverageTargets`, `setCoverageTargets`
- **Roles & permissions**: `getPermissionCatalogue`, `getProjectRoles`, `upsertProjectRole`, `deleteProjectRole`, `assignUserRoles`, `bulkAssignUserRoles`, `getMyPermissions`, `canAccess`, `getRoleAuditLog`, `getUserAssignments`, `removeUserAssignment`

See the [API Reference](./API-Reference.md) for payloads, responses and authorization rules. There is no generic "make any Jira request" resolver; every Jira call is a purpose-built, validated resolver.

### 3. Scheduled Function (`src/personalDataReporting.js`)

Runs weekly. It reads the app's personal-data index (`personal-data-account:<accountId>` records maintained by `personalDataIndex.js`), calls `privacy.reportPersonalData()` from `@forge/api`, and:
- for accounts reported as **closed**, invokes `gdprService.deleteUserData()` to erase the account's data;
- for accounts reported as **updated**, refreshes cached display names.

This is what the `report:personal-data` scope is for. See [Security → Personal Data Reporting](./Security.md#personal-data-reporting-gdpr).

### 4. Data Models

#### TicketEvaluation (`src/types/index.ts`)
```typescript
interface TicketEvaluation {
  id: string;                      // unique evaluation id
  ticketId: string;                // Jira issue id
  ticketKey: string;               // e.g. "PROJ-123"
  evaluatorId: string;             // Atlassian accountId of the evaluator
  evaluatorDisplayName?: string;
  timestamp: string;               // ISO 8601
  criteria: Record<string, number>;// criterionId -> score (0-10)
  averageScore: number;            // mean of criteria scores
  feedback?: string;               // optional free text
  version?: number;                // optimistic-concurrency counter
}
```
Ticket metadata (priority, issue type, status, assignee, created/resolved dates) is **not** stored on the evaluation; dashboards fetch it from Jira at read time via `getTicketMetadataBatch` and join on `ticketKey`.

#### EvaluationCriteria
```typescript
interface EvaluationCriteria {
  id: string;
  name: string;
  description: string;
  required: boolean;
  defaultValue: number;            // 1-10, default 5
}
```
Criteria are stored as an **ordered array**. Default set (8 criteria): Clarity of Description, Completeness of Requirements, Reproducibility (for bugs), Technical Documentation, Test Coverage, Code Quality, Solution Efficiency, Adherence to Standards. Weights are not supported; all criteria contribute equally to `averageScore`.

#### Storage Keys (Forge KV)

| Key | Content | Access path |
|-----|---------|-------------|
| `ticket-evaluations` | Global array of all evaluations | generic read; write requires `ticket.evaluate` |
| `project-evaluations-<KEY>` | Project-scoped array of evaluations | generic read (project must be allowed); write requires `ticket.evaluate` |
| `evaluation-criteria` | Ordered criteria array | generic read; first seed by anyone, edits admin-only |
| `coverage-targets` | `{ overallTarget, byPriorityTargets }` | dedicated resolvers (set = admin) |
| `theme-preference` | Theme choice | generic read/write |
| `selected-project` | Last selected project | generic read/write/delete |
| `user-displayname-cache` | accountId → display name | generic read/write |
| `evaluator-name-mapping` | Manual evaluator name overrides | generic read; write via `setEvaluatorName` (admin) |
| `evaluator-name-removed` | IDs whose override was removed | backend only |
| `removed-missing-evaluations` | Tickets hidden from the Missing Evaluations table | generic read/write |
| `allowed-projects`, `allowed-projects-audit` | Project allow-list and its change log | dedicated resolvers (set = admin) |
| `roles:<KEY>`, `role-assignments:<KEY>`, `role-audit:<KEY>` | RBAC definitions, assignments and audit trail | dedicated resolvers (`tab.settings` / admin) |
| `audit-log:<ts>:<id>`, `audit-index:<date>`, `security-alert:<ts>` | Backend audit events | backend only |
| `personal-data-account:<accountId>` | Personal-data index record | backend only |
| `gdpr-consent:<accountId>` | Consent record (helper exists, not used by any UI) | backend only |
| `user-cache-<id>`, `user-cache:<id>`, `evaluator-name:<id>`, `assignable-users-cache:<KEY>` | Server-side caches | backend only |
| `settings-admin-group` | Optional Jira group name granting admin rights (no UI; set manually) | backend only |

Keys outside the generic allow-lists are rejected by `getStorageValue` / `setStorageValue` / `deleteStorageValue` and the attempt is logged as `ACCESS_DENIED`.

## Data Flow

### 1. Evaluation Creation
```
Issue panel → EvaluationForm (sliders + feedback)
  → evaluationService.saveEvaluation()
      → invoke('setStorageValue', { key: 'ticket-evaluations', value: [...all, new] })
      → invoke('setStorageValue', { key: 'project-evaluations-<KEY>', value: [...project, new] })
  → backend checks ticket.evaluate for the ticket's project, records accountIds in the
    personal-data index, writes to Forge storage
  → cache invalidation → toast "Evaluation Saved"
```
Every save creates a **new** `TicketEvaluation` record; there is no in-place edit. Dashboards use the latest evaluation per ticket.

### 2. Analytics Generation
```
Dashboard tab → analyticsService / advancedAnalyticsService
  → in-memory cache? → IndexedDB persisted cache? (serve stale, refresh in background)
  → invoke('getStorageValue' | 'getFilteredEvaluations')          # evaluations
  → invoke('searchIssues' | 'getTicketMetadataBatch')             # Jira metadata (status, priority, type, assignee)
  → join on ticketKey → compute (Web Worker for advanced analytics) → render (Recharts)
```

### 3. Export
```
Export button → exportService.export*(format)
  → CSV: build rows, sanitise cell values (CSV-injection prefixing), Blob download
  → Excel: ExcelJS workbook with multiple worksheets, Blob download
```
File names follow `<dashboard>-<projectKey>-<yyyy-MM-dd>.csv|xlsx`.

## Integration Points

### Jira REST API v3 (via `@forge/api`)
| Endpoint | Used by | Auth |
|----------|---------|------|
| `GET /rest/api/3/issue/{idOrKey}` | `getIssue` | asUser |
| `POST /rest/api/3/search` (falls back to `POST /rest/api/3/search/jql` with `nextPageToken` on HTTP 410) | `searchIssues` (asUser), `getTicketMetadataBatch` (asApp, after a BROWSE_PROJECTS check as the user) | asUser / asApp |
| `GET /rest/api/3/myself` | `getCurrentUser` | asUser |
| `GET /rest/api/3/mypermissions?permissions=ADMINISTER` | `checkAdminPermission` | asUser |
| `GET /rest/api/3/mypermissions?projectKey=...&permissions=BROWSE_PROJECTS` | `getTicketMetadataBatch` | asUser |
| `GET /rest/api/3/user?accountId=...`, `GET /rest/api/3/user/search?accountId=...` | `getUser`, `batchGetUsers`, user caches, Personal Data Reporting | asApp |
| `GET /rest/api/3/user/assignable/search?project=...`, `GET /rest/api/3/user/search?query=...` | `searchUsers` (24 h per-project cache) | asApp |
| `GET /rest/api/3/project/{key}` | `getAllowedProjects` (avatar lookup) | asApp |
| `GET /rest/api/3/project/search?expand=insight&maxResults=50` | `searchProjects` | asUser |
| `GET /rest/api/3/serverInfo` | `getJiraBaseUrl` | asUser |
| `privacy.reportPersonalData()` (Forge API) | `personalDataReporting.js` | app |

`searchIssues` always prefixes the JQL with `project in (<allowed projects>)`; if no projects are allowed it returns an empty result set.

## Authorization Model

1. **Jira administrators** (users with the global `ADMINISTER` permission, or members of the optional `settings-admin-group` group) hold every permission in every project.
2. Everyone else receives permissions from the **roles assigned to them in the current project** (Settings → Roles & Access).
3. Permission catalogue: `dashboard.access`, `tab.analytics`, `tab.agentPerformance`, `tab.evaluatorAnalytics`, `tab.advancedAnalytics`, `tab.configuration` (Metrics tab), `tab.settings`, `ticket.evaluate`, `evaluation.delete`.
4. Checks are enforced **server-side** in the resolvers (`computeEffectivePermissions`), not only in the UI.

Details: [Security Documentation](./Security.md#authentication--authorization).

## Performance

### Caching
| Layer | Location | TTLs |
|-------|----------|------|
| In-memory (`cacheService`) | browser tab | SHORT 1 min, MEDIUM 5 min, LONG 30 min, VERY_LONG 2 h |
| Persisted (`persistedCache`, IndexedDB `qc-persisted-cache`) | browser profile | advanced analytics results (VERY_LONG), ticket metadata (LONG) |
| Server caches | Forge storage | user info (`user-cache-*`, 1 h), assignable users per project |

### Request management (`services/utils/api.ts`)
- Global concurrency limiter: at most **8** in-flight `invoke()` calls.
- In-flight deduplication: identical calls share one promise.
- `withRetry`: up to 3 attempts with **linear** back-off (`delay × attempt`). There is no special handling of HTTP 429; Forge/Jira platform limits apply.
- Prefetch hooks (`usePrefetch`, `usePrefetchAdvancedAnalytics`) warm adjacent tabs; the heaviest computations run in the Web Worker.

### Known limits
- Evaluations are stored as whole JSON arrays per key, and Forge KV enforces a per-value size limit. At roughly 400–700 bytes per evaluation, a key holds on the order of a few hundred evaluations before writes start failing; the global `ticket-evaluations` key is hit first because it aggregates every project. See [Entity Storage Migration Plan](./Entity-Storage-Migration-Plan.md) for the proposed remedy.
- Jira searches are capped per request (`maxResults` ≤ 100) and client code caps total issues fetched (e.g. 500 for the Missing Evaluations table).

## Security Architecture (summary)

- **Input validation** (`src/validators/inputValidator.js`): JQL (≤ 5000 chars, forbidden `;`, `--`, `/*`, `\\`, control chars), project key `^[A-Z][A-Z0-9_-]{1,9}$`, issue id, account id `^[a-zA-Z0-9:_-]{1,255}$`, storage key `^[a-zA-Z0-9:_-]{1,200}$`, pagination (`startAt` 0–100000, `maxResults` 1–100).
- **Authorization**: server-side permission checks and storage-key allow-lists (above).
- **Audit logging** (`auditService.js`): `ACCESS_DENIED` and `PERMISSION_CHANGED` events are written to `audit-log:*`; RBAC changes also go to `role-audit:<KEY>`.
- **Data protection**: Forge storage is encrypted at rest by Atlassian; all traffic is HTTPS; the app never stores emails, passwords or tokens.
- **Personal data**: weekly Personal Data Reporting with automatic erasure of closed accounts.

Full details in [Security Documentation](./Security.md).

## Deployment

```yaml
# manifest.yml (abridged; see repository for the full file)
modules:
  jira:globalPage:
    - key: ticket-evaluation-dashboard
      resource: main
      resolver: { function: resolver }
  jira:issuePanel:
    - key: ticket-evaluation-panel
      resource: issue-panel
      resolver: { function: resolver }
  scheduledTrigger:
    - key: personal-data-reporting-weekly
      function: personalDataReporter
      interval: week
  function:
    - key: resolver
      handler: index.handler
    - key: personalDataReporter
      handler: personalDataReporting.handler
resources:
  - key: main
    path: dist
  - key: issue-panel
    path: dist
```

Build pipeline:
1. `npm run build` → Vite builds the Custom UI into `dist/` (workers created with `new Worker(new URL('./workers/advancedAnalyticsWorker.ts', import.meta.url), { type: 'module' })`; `vite.config.ts` sets `worker.format = 'es'`, `assetsInlineLimit = 0` for CSP compliance, and manual vendor chunks).
2. `forge deploy -e <environment>` uploads `dist/` and the `src/*.js` functions.
3. `forge install` / `forge install --upgrade` installs onto a site (required when scopes change).

Environments: `development`, `staging`, `production`. Production is the environment behind the Atlassian Marketplace listing.

## Appendix: Repository Layout

```
quality-scorecard-for-jira/
├── manifest.yml
├── package.json                  # version 1.3.0
├── vite.config.ts
├── index.html
├── src/
│   ├── index.js                  # Forge resolvers
│   ├── personalDataReporting.js  # Scheduled trigger handler
│   ├── main.tsx / App.tsx
│   ├── pages/
│   ├── components/
│   ├── contexts/
│   ├── hooks/
│   ├── services/
│   ├── types/
│   ├── validators/
│   ├── middleware/rateLimiter.js # legacy, not wired in
│   └── utils/errorHandler.js     # legacy, not wired in
├── dist/                         # build output (Forge resource)
└── Documentation/                # developer copy of these docs
```

## See Also

- [Installation Guide](./Installation.md)
- [API Reference](./API-Reference.md)
- [Development Guide](./Development.md)
- [Security Documentation](./Security.md)
- [Technical Troubleshooting](./Troubleshooting-Technical.md)
- [Entity Storage Migration Plan](./Entity-Storage-Migration-Plan.md)

---

*Last Updated: October 2026 — reflects app version 1.3.0 (production deployment 7.x)*
