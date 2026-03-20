# Forge Entity Storage Migration Plan

**Status:** Planned
**Priority:** High — required before enterprise adoption
**Estimated Effort:** 3–5 days

---

## 1. Problem Statement

### Current Architecture

All evaluations are stored as a single JSON array under two Key-Value Storage keys:

| Key | Content | Scope |
|-----|---------|-------|
| `ticket-evaluations` | Array of **all** evaluations across all projects | Global |
| `project-evaluations-{projectKey}` | Array of evaluations for a single project | Per-project |

### Why This Breaks at Scale

Forge Key-Value Storage enforces a **200KB limit per key**. A single evaluation record is approximately **400–700 bytes** (varies with feedback length).

| Evaluations per key | Approx. size | Status |
|---------------------|-------------|--------|
| 100 | ~50 KB | Safe |
| 300 | ~150 KB | Warning zone |
| 400 | ~200 KB | **Limit reached** |
| 500+ | >200 KB | **Write failures** |

**Enterprise impact:** A team evaluating 10 tickets/day hits the global key limit in ~40 days. Multi-project organizations hit it sooner. Once the limit is reached, `storage.set()` silently fails or throws, and no new evaluations can be saved.

### Additional Problems with Current Approach

1. **No server-side filtering** — Every query loads the full array into memory, then filters client-side
2. **Read-modify-write race conditions** — Concurrent evaluations can overwrite each other
3. **No pagination** — All data must be loaded at once
4. **O(n) scan for every query** — Performance degrades linearly with evaluation count

---

## 2. Target Architecture: Entity Storage

Forge Entity Storage stores each evaluation as an **individual entity** (one record per key) with typed, indexed attributes that support server-side filtering and cursor-based pagination.

### 2.1 Entity Schema

```yaml
# manifest.yml addition
app:
  storage:
    entities:
      - name: evaluation
        attributes:
          projectKey:
            type: string
          ticketId:
            type: string
          ticketKey:
            type: string
          evaluatorId:
            type: string
          timestamp:
            type: string
          averageScore:
            type: float
          version:
            type: integer
        indexes:
          - name: by-project-timestamp
            partition:
              - projectKey
            range:
              - timestamp
          - name: by-project-evaluator
            partition:
              - projectKey
            range:
              - evaluatorId
          - name: by-ticket
            partition:
              - ticketKey
            range:
              - timestamp
          - name: by-evaluator-timestamp
            partition:
              - evaluatorId
            range:
              - timestamp
          - name: by-project-score
            partition:
              - projectKey
            range:
              - averageScore
```

### 2.2 Entity Key Format

Each evaluation gets a unique key:

```
eval:{projectKey}:{ticketKey}:{evaluatorId}:{timestamp}
```

Example: `eval:PROJ:PROJ-123:5af7d3b2e4b0c:2026-03-07T14:30:00Z`

### 2.3 Entity Value

The full evaluation payload (criteria scores, feedback, evaluator display name) is stored as the entity value. Attributes are extracted for indexing; the full object is returned on read.

```typescript
interface EvaluationEntity {
  // Indexed attributes (also in manifest)
  projectKey: string;
  ticketId: string;
  ticketKey: string;
  evaluatorId: string;
  timestamp: string;       // ISO 8601
  averageScore: number;
  version: number;

  // Non-indexed data (stored in value, not queryable)
  evaluatorDisplayName: string;
  criteria: Record<string, number>;
  feedback: string;
  id: string;              // Legacy ID for backward compatibility
}
```

### 2.4 Index Design Rationale

| Index | Query Pattern | Used By |
|-------|--------------|---------|
| `by-project-timestamp` | "All evaluations for project X in date range" | Analytics dashboard, coverage analysis, agent performance |
| `by-project-evaluator` | "All evaluations by evaluator Y in project X" | Evaluator analytics, GDPR data export |
| `by-ticket` | "All evaluations for ticket PROJ-123" | Evaluation history, issue panel |
| `by-evaluator-timestamp` | "All evaluations by user Z (cross-project)" | GDPR export/delete, user profile |
| `by-project-score` | "Evaluations in project X with score > 7" | Advanced analytics, quality trending |

5 indexes used out of 7 maximum — leaves 2 slots for future needs.

---

## 3. Migration Strategy

### 3.1 Approach: Dual-Write with Gradual Read Migration

The migration follows a three-phase approach to ensure zero downtime and backward compatibility.

```
Phase 1: Dual-Write        Phase 2: Read Migration      Phase 3: Cleanup
┌──────────────────┐       ┌──────────────────┐         ┌──────────────────┐
│ Write: KV + Entity│       │ Read: Entity first│         │ Write: Entity only│
│ Read:  KV only    │       │ Fallback: KV      │         │ Read:  Entity only│
│                   │       │                   │         │ Delete: KV keys   │
└──────────────────┘       └──────────────────┘         └──────────────────┘
```

### 3.2 Phase 1 — Dual-Write (Safe, No Breaking Changes)

**Goal:** Start populating Entity Storage alongside existing KV storage. Reads still come from KV.

**Changes:**

1. **Add entity definitions to `manifest.yml`** (see Section 2.1)
2. **Update `setStorageValue` resolver** — When writing evaluations, also write each evaluation as an individual entity
3. **Update `saveEvaluation` in `evaluationService.ts`** — After saving the array to KV, also upsert the individual entity
4. **Create a one-time migration resolver** (`migrateToEntityStorage`) — Reads all existing evaluations from KV and writes each as an entity

```typescript
// Pseudocode for dual-write in the resolver
resolver.define('saveEvaluation', async (req) => {
  const evaluation = req.payload;

  // Existing KV write (unchanged)
  const allEvals = await storage.get('ticket-evaluations') || [];
  allEvals.push(evaluation);
  await storage.set('ticket-evaluations', allEvals);

  // New Entity write
  const entityKey = `eval:${evaluation.ticketKey}:${evaluation.evaluatorId}:${evaluation.timestamp}`;
  await storage.entity('evaluation').set(entityKey, {
    projectKey: evaluation.ticketKey.split('-')[0],
    ticketId: evaluation.ticketId,
    ticketKey: evaluation.ticketKey,
    evaluatorId: evaluation.evaluatorId,
    timestamp: evaluation.timestamp,
    averageScore: evaluation.averageScore,
    version: evaluation.version,
    // Full payload
    evaluatorDisplayName: evaluation.evaluatorDisplayName,
    criteria: evaluation.criteria,
    feedback: evaluation.feedback,
    id: evaluation.id,
  });
});
```

### 3.3 Phase 2 — Read Migration

**Goal:** Switch reads to Entity Storage with KV fallback.

**New resolvers to add:**

```typescript
resolver.define('queryEvaluations', async (req) => {
  const { projectKey, evaluatorId, startDate, endDate, cursor, limit } = req.payload;

  let query = storage.entity('evaluation').query()
    .index('by-project-timestamp')
    .where('projectKey', projectKey);

  if (startDate) query = query.where('timestamp', startAfter(startDate));
  if (endDate) query = query.where('timestamp', conditions.before(endDate));
  if (limit) query = query.limit(limit);
  if (cursor) query = query.cursor(cursor);

  const results = await query.getMany();
  return {
    evaluations: results.results.map(r => r.value),
    cursor: results.nextCursor,
  };
});

resolver.define('getEvaluationsByTicket', async (req) => {
  const { ticketKey } = req.payload;

  const results = await storage.entity('evaluation').query()
    .index('by-ticket')
    .where('ticketKey', ticketKey)
    .sort('DESC')
    .getMany();

  return results.results.map(r => r.value);
});

resolver.define('getEvaluationsByEvaluator', async (req) => {
  const { evaluatorId, startDate, endDate } = req.payload;

  let query = storage.entity('evaluation').query()
    .index('by-evaluator-timestamp')
    .where('evaluatorId', evaluatorId);

  if (startDate) query = query.where('timestamp', startAfter(startDate));
  if (endDate) query = query.where('timestamp', conditions.before(endDate));

  const results = await query.getMany();
  return results.results.map(r => r.value);
});
```

**Frontend changes:**
- `evaluationService.ts` — Use `queryEvaluations` resolver instead of loading full arrays
- `analyticsService.ts` — Paginate through evaluations using cursors
- `advancedAnalyticsService.ts` — Same cursor-based pagination

### 3.4 Phase 3 — Cleanup

**Goal:** Remove KV storage dependency entirely.

1. Remove dual-write logic from save resolver
2. Remove KV read fallback
3. Delete legacy KV keys (`ticket-evaluations`, `project-evaluations-*`)
4. Remove array manipulation code from `evaluationService.ts`

---

## 4. Manifest Changes

### Before (current)

```yaml
permissions:
  scopes:
    - read:jira-work
    - write:jira-work
    - storage:app
    - read:jira-user
```

### After (with Entity Storage)

```yaml
permissions:
  scopes:
    - read:jira-work
    - write:jira-work
    - storage:app
    - read:jira-user

app:
  storage:
    entities:
      - name: evaluation
        attributes:
          projectKey:
            type: string
          ticketId:
            type: string
          ticketKey:
            type: string
          evaluatorId:
            type: string
          timestamp:
            type: string
          averageScore:
            type: float
          version:
            type: integer
        indexes:
          - name: by-project-timestamp
            partition:
              - projectKey
            range:
              - timestamp
          - name: by-project-evaluator
            partition:
              - projectKey
            range:
              - evaluatorId
          - name: by-ticket
            partition:
              - ticketKey
            range:
              - timestamp
          - name: by-evaluator-timestamp
            partition:
              - evaluatorId
            range:
              - timestamp
          - name: by-project-score
            partition:
              - projectKey
            range:
              - averageScore
```

**Note:** The `storage:app` scope already covers Entity Storage — no additional scopes are needed.

---

## 5. Resolver Changes Summary

| Resolver | Current | After Migration |
|----------|---------|-----------------|
| `getStorageValue` (for evaluations) | `storage.get('ticket-evaluations')` → full array | Replaced by `queryEvaluations` |
| `setStorageValue` (for evaluations) | Read-modify-write full array | `storage.entity('evaluation').set(key, value)` |
| `getFilteredEvaluations` | Load all, filter in memory | Server-side index queries |
| `deleteTicketEvaluations` | Load array, filter out, save | `storage.entity('evaluation').delete(key)` per entity |
| **New:** `queryEvaluations` | N/A | Paginated, indexed query by project + date |
| **New:** `getEvaluationsByTicket` | N/A | Indexed query by ticket key |
| **New:** `getEvaluationsByEvaluator` | N/A | Indexed query by evaluator (GDPR) |
| **New:** `migrateToEntityStorage` | N/A | One-time migration of existing KV data |

---

## 6. Frontend Service Changes

### evaluationService.ts

| Function | Current | After |
|----------|---------|-------|
| `getAllEvaluations(projectKey)` | `forgeStorage.get('ticket-evaluations')` + filter | `invoke('queryEvaluations', { projectKey, limit: 100 })` with cursor pagination |
| `getEvaluationHistory(ticketId)` | Load all, filter, slice | `invoke('getEvaluationsByTicket', { ticketKey })` |
| `getExistingEvaluation(ticketId, evaluatorId)` | Load all, find match | `invoke('getEvaluationsByTicket', { ticketKey })` + client filter by evaluatorId |
| `saveEvaluation(...)` | Load array, push, save | `invoke('saveEvaluation', evaluation)` — backend handles entity write |

### analyticsService.ts

| Function | Current | After |
|----------|---------|-------|
| `getAnalyticsData(filters)` | Load all evaluations, compute in memory | Paginate with `queryEvaluations` cursor, compute incrementally or use web worker |
| `getAgentPerformanceData(filters)` | Load all, group by assignee | Same paginated approach |
| `getEvaluatorAnalyticsData(filters)` | Load all, group by evaluator | Use `by-project-evaluator` index |

### gdprService.js

| Function | Current | After |
|----------|---------|-------|
| `exportUserData(accountId)` | Load all evaluations, filter | `invoke('getEvaluationsByEvaluator', { evaluatorId })` |
| `deleteUserData(accountId)` | Load all, remove matches, save | Query by evaluator, delete each entity |

---

## 7. Data Migration Approach

### One-Time Migration Resolver

```typescript
resolver.define('migrateToEntityStorage', async (req) => {
  const isAdmin = await checkAdminPermission(req);
  if (!isAdmin) throw new Error('Admin only');

  const allEvaluations = await storage.get('ticket-evaluations') || [];
  let migrated = 0;
  let skipped = 0;

  for (const evaluation of allEvaluations) {
    const projectKey = evaluation.ticketKey?.split('-')[0];
    if (!projectKey || !evaluation.ticketKey || !evaluation.evaluatorId) {
      skipped++;
      continue;
    }

    const entityKey = `eval:${evaluation.ticketKey}:${evaluation.evaluatorId}:${evaluation.timestamp}`;

    await storage.entity('evaluation').set(entityKey, {
      projectKey,
      ticketId: evaluation.ticketId,
      ticketKey: evaluation.ticketKey,
      evaluatorId: evaluation.evaluatorId,
      timestamp: evaluation.timestamp,
      averageScore: evaluation.averageScore,
      version: evaluation.version || 1,
      evaluatorDisplayName: evaluation.evaluatorDisplayName,
      criteria: evaluation.criteria,
      feedback: evaluation.feedback,
      id: evaluation.id,
    });

    migrated++;
  }

  return { migrated, skipped, total: allEvaluations.length };
});
```

### Migration Trigger

Add a one-time migration check that runs on app load (admin only):

1. Check for a flag: `storage.get('entity-migration-complete')`
2. If not set, prompt admin to run migration
3. After successful migration, set the flag

---

## 8. Capacity After Migration

| Metric | KV Storage (current) | Entity Storage (after) |
|--------|---------------------|----------------------|
| Max evaluations per project | ~400 | **Unlimited** (individual entities) |
| Max total evaluations | ~400 | **Unlimited** |
| Query performance at 10K evals | O(n) full scan | O(log n) indexed lookup |
| Pagination | Client-side slice | Server-side cursor |
| Concurrent writes | Race condition risk | Atomic per-entity writes |
| Max entity value size | N/A | 200KB per entity (single evaluation is ~500 bytes — not a concern) |

---

## 9. Risks and Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Entity/index deletion is permanent in production | Cannot remove deployed attributes | Design schema carefully; keep 2 index slots reserved |
| Index creation time scales with data | Slow deploys for large datasets | Run migration during low-traffic window; monitor with `forge storage entities indexes list` |
| Backward compatibility during dual-write | Temporary data duplication | Phase 2 reads from Entity first, KV fallback; Phase 3 removes duplication |
| Query result limits | `getMany()` returns max 1000 results per call | Implement cursor-based pagination in all query paths |
| Frontend analytics computation | Large datasets still need aggregation | Move aggregation to web worker (already partially implemented in `advancedAnalyticsWorker.ts`) |

---

## 10. Implementation Checklist

### Phase 1: Dual-Write
- [ ] Add entity definition to `manifest.yml`
- [ ] Deploy to development environment
- [ ] Wait for index creation (`forge storage entities indexes list`)
- [ ] Add `migrateToEntityStorage` resolver
- [ ] Add dual-write to `saveEvaluation` resolver
- [ ] Add dual-write to `deleteTicketEvaluations` resolver
- [ ] Run migration on development site
- [ ] Verify entity data matches KV data

### Phase 2: Read Migration
- [ ] Add `queryEvaluations` resolver
- [ ] Add `getEvaluationsByTicket` resolver
- [ ] Add `getEvaluationsByEvaluator` resolver
- [ ] Update `evaluationService.ts` to use new resolvers
- [ ] Update `analyticsService.ts` with cursor pagination
- [ ] Update `advancedAnalyticsService.ts` with cursor pagination
- [ ] Update `gdprService.js` to use evaluator index
- [ ] Test all analytics dashboards with Entity Storage reads
- [ ] Deploy to staging; validate with real data volume

### Phase 3: Cleanup
- [ ] Remove KV read fallback
- [ ] Remove dual-write (Entity-only writes)
- [ ] Delete legacy KV keys via admin resolver
- [ ] Remove array manipulation code
- [ ] Deploy to production

---

*Document Version: 1.0 | Created: March 2026*
