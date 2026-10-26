# Security Documentation

## Quick Navigation
- [Architecture Overview](./Architecture.md#authorization-model) - Where security sits in the design
- [API Reference](./API-Reference.md) - Authorization rule for every resolver
- [Installation Guide](./Installation.md#step-3-grant-access) - Setting up access
- [Development Guide](./Development.md#security-practices) - Secure coding practices
- [Technical Troubleshooting](./Troubleshooting-Technical.md#permission-and-access-issues) - Access problems

## Security Overview

Quality Scorecard for Jira runs entirely on Atlassian Forge. Atlassian operates the hosting, network, TLS termination, storage encryption and tenant isolation; the app is responsible for authorization logic, input validation, what it stores, and how it handles personal data. This document describes exactly what the app implements. Where a control is provided by the platform rather than the app, it says so.

## Security Architecture

```
┌──────────────────────────────────────────────────────────────┐
│ Atlassian / Forge platform (not app code)                    │
│  • Atlassian account authentication (SSO/MFA per site)       │
│  • TLS in transit, encryption at rest for Forge storage      │
│  • Per-installation tenant isolation of storage              │
│  • Custom UI sandboxing and Forge CSP                        │
│  • Invocation and REST API rate limits                       │
├──────────────────────────────────────────────────────────────┤
│ App backend – src/index.js resolvers                         │
│  • Server-side permission checks (RBAC + Jira admin)         │
│  • Storage-key allow-lists for generic storage resolvers     │
│  • Input validation (inputValidator.js)                      │
│  • Project allow-list enforced in every JQL search           │
│  • Audit logging of denied access and permission changes     │
├──────────────────────────────────────────────────────────────┤
│ Scheduled job – src/personalDataReporting.js                 │
│  • Weekly Personal Data Reporting to Atlassian               │
│  • Automatic erasure for closed accounts                     │
├──────────────────────────────────────────────────────────────┤
│ App frontend – React Custom UI                               │
│  • Tabs/actions hidden without permission (UX only)          │
│  • CSV-injection sanitisation on export                      │
└──────────────────────────────────────────────────────────────┘
```

## Authentication & Authorization

### Authentication
Users are authenticated by Jira Cloud. Forge passes the caller's `accountId` into every resolver (`req.context.accountId`); the app never handles credentials, sessions or tokens.

### Requested scopes (`manifest.yml`)

```yaml
permissions:
  scopes:
    - read:jira-work        # read issues/projects (search, issue, serverInfo)
    - storage:app           # Forge Key-Value storage
    - read:jira-user        # resolve account ids to display names
    - report:personal-data  # Personal Data Reporting API
```

The app does **not** request `write:jira-work`. It never creates, edits or transitions Jira issues.

### Authorization model

| Principal | How determined | Effective permissions |
|-----------|----------------|-----------------------|
| Jira administrator | `GET /rest/api/3/mypermissions?permissions=ADMINISTER` returns `havePermission: true`, **or** the caller is a member of the Jira group named in the `settings-admin-group` storage key (optional, no UI) | All permissions in every project |
| Everyone else | Roles assigned to the account in `role-assignments:<PROJECT>` and the permissions of those roles in `roles:<PROJECT>` | Union of role permissions, per project |

Permission catalogue: `dashboard.access`, `tab.analytics`, `tab.agentPerformance`, `tab.evaluatorAnalytics`, `tab.advancedAnalytics`, `tab.configuration`, `tab.settings`, `ticket.evaluate`, `evaluation.delete`.

Roles are created and assigned by Jira administrators in **Settings → Roles & Access**. A user with no role in a project cannot open that project's dashboard (`dashboard.access`) or save evaluations (`ticket.evaluate`).

### Server-side enforcement
The UI hides tabs and buttons the user is not entitled to, but every sensitive operation is re-checked in the resolver:

| Operation | Check |
|-----------|-------|
| Save evaluation (`setStorageValue` on `ticket-evaluations` / `project-evaluations-*`) | `ticket.evaluate` in the ticket's project; clearing the global list requires admin |
| Delete a ticket's evaluations | `evaluation.delete` |
| Edit criteria (`evaluation-criteria`, once seeded) | admin |
| Project allow-list, coverage targets, role definitions, role assignments, evaluator-name overrides | admin |
| Read role assignments / role audit log | `tab.settings` |
| Fetch ticket metadata (`getTicketMetadataBatch`) | project in allow-list **and** Jira `BROWSE_PROJECTS` verified as the user |
| Any other storage key through the generic resolvers | rejected (allow-lists: 9 readable, 4 writable, 1 deletable key) |

Denied attempts are written to the audit log as `ACCESS_DENIED`.

### Project isolation
`searchIssues` wraps every JQL as `project in (<allowed projects>) AND (...)`. If the allow-list is empty, no Jira search is executed. Jira's own permission scheme still applies because searches run `asUser`.

## Input Validation

Implemented in `src/validators/inputValidator.js` and applied in the resolvers:

| Input | Rule |
|-------|------|
| JQL | non-empty string, ≤ 5000 characters; rejects `;`, `--`, `/*`, `\\` and ASCII control characters |
| Project key | `^[A-Z][A-Z0-9_-]{1,9}$` |
| Issue id/key | `^([A-Z][A-Z0-9_-]{1,9}-)?[0-9]{1,10}$` |
| Account id | `^[a-zA-Z0-9:_-]{1,255}$` |
| Storage key | `^[a-zA-Z0-9:_-]{1,200}$` |
| Pagination | `startAt` 0–100 000, `maxResults` 1–100 |
| Free text (`sanitizeString`) | control characters removed, trimmed, truncated to 10 000 characters |

Scores are constrained by the UI slider (integers 0–10). Feedback is stored verbatim and rendered as text by React (no HTML injection); it is not length-limited by the backend.

### CSV export hardening
`exportService.ts` (`sanitizeCSVValue`) prefixes CSV cell values that start with `=`, `+`, `-` or `@` with an apostrophe and quotes values containing commas or quotes, so spreadsheets do not interpret them as formulas. The Excel exports do not call this function; ExcelJS writes the values as typed string cells, which spreadsheet applications treat as text rather than formulas.

## Data Protection

### What the app stores
| Category | Keys | Personal data? |
|----------|------|----------------|
| Evaluations | `ticket-evaluations`, `project-evaluations-<KEY>` | evaluator `accountId`, display name, free-text feedback |
| Criteria & targets | `evaluation-criteria`, `coverage-targets` | no |
| Access configuration | `allowed-projects`, `allowed-projects-audit`, `roles:*`, `role-assignments:*`, `role-audit:*` | account ids and display names of admins/assignees |
| Preferences | `theme-preference`, `selected-project` | no |
| Name caches | `user-displayname-cache`, `evaluator-name-mapping`, `evaluator-name-removed`, `user-cache*`, `evaluator-name:*`, `assignable-users-cache:*` | account id → display name |
| Audit | `audit-log:*`, `audit-index:*`, `security-alert:*` | actor account ids |
| Personal-data index | `personal-data-account:<id>` | account id + last refresh time |

The app does **not** store e-mail addresses, passwords, API tokens, IP addresses or issue content beyond what Jira returns at read time for display.

### Encryption
- **At rest**: Forge storage is encrypted by Atlassian (platform control).
- **In transit**: all calls are HTTPS; the Custom UI talks to the backend only through the Forge bridge.
- No data leaves Atlassian infrastructure: the manifest declares no `external` egress permissions.

### Browser-side data
The frontend keeps analytics results and ticket metadata in an IndexedDB database (`qc-persisted-cache`) and an in-memory cache for performance. This data is scoped to the user's browser profile and the Jira site origin; it contains the same evaluation/ticket data the user is already authorised to see.

## Audit Logging

`src/services/auditService.js` is wired into the resolvers. Events currently emitted:

| Event | When |
|-------|------|
| `ACCESS_DENIED` | generic storage read/write/delete outside the allow-list; evaluation write without `ticket.evaluate`; evaluation delete without `evaluation.delete`; reading assignments/audit log without `tab.settings` |
| `PERMISSION_CHANGED` | role created/updated/deleted, roles assigned (single/bulk), user access removed, evaluations deleted |
| `USER_DATA_DELETED` | Personal Data Reporting erased a closed account |

Entry format (stored at `audit-log:<timestamp>:<id>`, indexed per day in `audit-index:<yyyy-mm-dd>`):

```json
{
  "id": "…",
  "timestamp": "2026-10-25T10:30:00.000Z",
  "event": "ACCESS_DENIED",
  "severity": "WARNING",
  "accountId": "…",
  "projectKey": "PROJ",
  "ipAddress": null,
  "userAgent": null,
  "details": { "action": "storage_write_denied", "key": "roles:PROJ", "requestId": null }
}
```
`ipAddress` and `userAgent` are always `null`: Forge does not expose them to resolvers.

In addition:
- Project allow-list changes are kept in `allowed-projects-audit` and shown in **Settings → Project Access & Recent Changes**.
- Role/permission changes are kept in `role-audit:<PROJECT>` and shown in **Settings → Roles & Access** (requires `tab.settings`).

Retention: the service defines a 90-day retention for `audit-log:*` entries, but the clean-up routine is currently a no-op; audit entries persist until the app is uninstalled. The `audit-log:*` records are not exposed in the UI; they can be inspected by the vendor via Forge tooling on request.

## Personal Data Reporting (GDPR)

Atlassian requires apps that cache personal data to report the account ids they hold so Atlassian can signal closed or updated accounts. The app implements this with the `report:personal-data` scope and a weekly `scheduledTrigger`:

1. `personalDataIndex.js` records every account id the app stores (evaluators, assignees, role assignees, name-mapping entries) in `personal-data-account:<id>` with an `updatedAt` timestamp, and periodically re-discovers ids from existing storage.
2. `personalDataReporting.js` calls `privacy.reportPersonalData()` with the full list.
3. Accounts reported as **closed** are passed to `gdprService.deleteUserData(accountId)`, which removes the account's evaluations from `ticket-evaluations`, deletes its entry in `evaluator-name-mapping`, removes `gdpr-consent:<id>` and the index record, and writes a `USER_DATA_DELETED` audit event.
4. Accounts reported as **updated** have their display name re-fetched from Jira and refreshed in `evaluator-name-mapping`.

Known limitation (tracked for a future release): `deleteUserData` does not yet purge the account from `project-evaluations-<KEY>` lists, role assignments, audit entries or display-name caches. Customers who need a complete erasure for a specific account should contact support (see [Privacy Policy](../Legal/Privacy-Policy.md)).

### Data subject requests
There is no self-service export/erasure UI. The vendor fulfils access, rectification and erasure requests on request from the customer's Jira administrator, using Forge tooling against the customer's installation. Customers can also export evaluation data at any time from the dashboards (CSV/Excel), and uninstalling the app removes all app storage for the site.

### Data retention
The app does **not** automatically expire evaluations, configuration or caches. `gdprService.js` contains retention constants (`RETENTION_PERIODS`) and an `enforceDataRetention()` helper, but no scheduled job calls it. Data is retained until:
- an administrator deletes a ticket's evaluations (`evaluation.delete`),
- Personal Data Reporting erases a closed account, or
- the app is uninstalled from the site.

## Vulnerability Management

- `npm run security-check` → `npm audit --audit-level=high`.
- An SBOM (`sbom.json`) is generated for each release.
- `forge lint` is run before deployment.
- Security-relevant dependencies (ExcelJS, Vite/esbuild, Radix) are kept current; the app has no server-side third-party services.

### Hardening history (production 7.x, 2026)
| Change | Effect |
|--------|--------|
| Removed generic `makeApiRequest` resolver and debug resolvers | Frontend can no longer call arbitrary Jira endpoints |
| Added storage-key allow-lists to generic storage resolvers | Roles, assignments, audit logs, consent and index records are unreachable from the client |
| `ticket.evaluate` required for evaluation writes; admin required for criteria edits | Prevents unauthenticated writes from a tampered client |
| BROWSE_PROJECTS check before `asApp` ticket metadata fetch | Prevents cross-project data exposure |
| Dropped `write:jira-work` scope | Least privilege |
| Audit logging on denied access | Detectability |

## Incident Response

Security incidents are handled under the vendor's internal Incident Response Plan (severity classification, containment, customer notification within 72 hours for personal-data breaches as required by GDPR, post-incident review). Report suspected vulnerabilities or incidents to **support@nexusfrontier.net** with "SECURITY" in the subject line. See also the [Cloud Security Statement](../Legal/Cloud-Security-Statement.md) and [Support SLA](../Legal/Support-SLA.md).

## Release Security Checklist (vendor)

- [ ] `npm audit --audit-level=high` clean or exceptions documented
- [ ] `forge lint` clean
- [ ] Scopes in `manifest.yml` unchanged, or change reviewed and documented
- [ ] No new generic storage keys added to allow-lists without an authorization rule
- [ ] All new resolvers validate inputs and check permissions server-side
- [ ] SBOM regenerated
- [ ] Documentation (this file, API Reference, Privacy Policy data inventory) updated

## Compliance Posture

- **GDPR**: the vendor acts as processor; see the [Data Processing Agreement](../Legal/Data-Processing-Agreement.md). Technical measures are those described above.
- **Hosting certifications**: provided by Atlassian for the Forge platform (SOC 2, ISO 27001 and others, per Atlassian's Trust Center). The app itself has not undergone an independent certification or third-party penetration test.
- **Marketplace**: the app is distributed through the Atlassian Marketplace and subject to Atlassian's app security requirements and review.

## Related Resources

- [Architecture Documentation](./Architecture.md)
- [API Reference](./API-Reference.md)
- [Installation Guide](./Installation.md)
- [Development Guide](./Development.md)
- [Technical Troubleshooting](./Troubleshooting-Technical.md)
- [Privacy Policy](../Legal/Privacy-Policy.md) · [Cloud Security Statement](../Legal/Cloud-Security-Statement.md)

---

*For security concerns or vulnerability reports, contact: support@nexusfrontier.net*

*Last Updated: October 2026 — reflects app version 1.3.0 (production deployment 7.x)*
