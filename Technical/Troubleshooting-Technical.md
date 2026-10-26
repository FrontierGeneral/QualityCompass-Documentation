# Technical Troubleshooting Guide

## Overview

Solutions for issues administrators and developers may encounter with Quality Scorecard for Jira. Every symptom below maps to behaviour that actually exists in the app; error strings are quoted from the source so they can be searched in `forge logs`.

## Quick Links
- [Architecture Overview](./Architecture.md)
- [API Reference](./API-Reference.md) - Resolver payloads and authorization rules
- [Installation Guide](./Installation.md) - First-run configuration
- [Development Guide](./Development.md) - Tunnel, logs, build
- [Security Documentation](./Security.md) - Permission model

---

## Installation and Visibility Issues

### The app is not in the Apps menu
- Confirm it is listed and **Enabled** under **Apps → Manage apps**.
- The global page is titled **Quality Scorecard for Jira**; it is under the **Apps** menu, not under project settings.
- Developers: `forge install list` shows which sites/environments have the app. The module key is `ticket-evaluation-dashboard` (global page) and `ticket-evaluation-panel` (issue panel).

### The issue panel does not appear on an issue
- The issue panel is rendered on every issue, but it only works for issues in **allowed projects**. If the project is not allowed, the permission check fails (`Access denied: Project <KEY> is not in allowed projects` in the logs) and **Save Evaluation** shows the toast "Not allowed – You do not have permission to evaluate this ticket."
- Some Jira issue layouts collapse app panels; look for **Quality Scorecard for Jira** in the panel list / "…" menu of the issue view.

### Landing page says "No projects are configured yet"
Expected on first run. An administrator must click **Configure Projects** and add at least one project. Non-admins see "Ask your administrator to add your projects to the QA app's allowed list."

---

## Permission and Access Issues

### A user cannot see the dashboard or some tabs
Access is granted per project through **Settings → Roles & Access**. Verify:
1. The user has a role in **this** project (roles are not shared across projects).
2. The role includes `Dashboard Access` plus the relevant tab permissions.
3. The user is not expecting admin behaviour – only users with Jira's global **Administer Jira** permission (or members of the optional `settings-admin-group` group) get everything automatically.

Useful check from the browser console inside the app iframe is not possible (Custom UI sandbox); instead have the user reload, or verify via `invoke('getMyPermissions', { projectKey })` in a development build.

### "Not allowed to evaluate tickets"
The user lacks `ticket.evaluate` (Evaluate Tickets) in the ticket's project. Assign a role that includes it.

### "Unauthorized: Missing evaluation.delete permission"
Deleting a ticket's evaluations requires the `Delete Evaluations` permission.

### "Only administrators can …"
Role definitions, role assignments, project allow-list, coverage targets and evaluator-name overrides are admin-only; `Settings` tab permission is not enough.

### "Storage key not permitted for read|write|delete"
The frontend asked the generic storage resolver for a key outside its allow-list. In a stock build this indicates a stale cached frontend bundle (hard-refresh) or a modified client. The attempt is also logged as an `ACCESS_DENIED` audit event.

### "Not allowed to view issues in this project"
`getTicketMetadataBatch` verified that the caller lacks Jira **Browse Projects** on the project. Fix the Jira permission scheme.

---

## Data and Analytics Issues

### Evaluations saved but not shown in dashboards
- Dashboards filter by **date range** and, for most cards, by the ticket's **created** or **resolved** date fetched from Jira — not by the evaluation timestamp. Widen the range and click **Apply Filters**.
- Analytics are cached (in-memory up to 30 min; Advanced Analytics up to 2 h in IndexedDB). Use **Refresh** on the Advanced Analytics tab or reload the page.
- The evaluation's `ticketKey` must start with the project key of the dashboard you are viewing.

### Missing Evaluations table is empty
The table requires you to pick a date range and click **Load**. It lists issues with `statusCategory = Done` created in the range (capped at 500) that have no evaluation. Rows removed with the **Remove** action are remembered in `removed-missing-evaluations`.

### Numbers differ between tabs
- Analytics Dashboard uses tickets by **created** date; Advanced Analytics coverage uses **resolution** date; Agent Performance uses the latest evaluation per ticket. Different bases, different totals.
- Evaluator Analytics "Biased Evaluators" flags evaluators whose average deviates from the team average by more than **2.5 points**.

### Evaluator shows as "Current User (xxxxxxxx)" or an account id
The display name could not be resolved. Causes: the `read:jira-user` scope was not granted on upgrade (`forge install --upgrade`), or the account is deactivated/closed. Names are resolved through `batchGetUsers` → Jira `GET /rest/api/3/user`, then cached in `user-displayname-cache`.

### "JQL query too long" / "Invalid JQL: contains forbidden characters"
Backend validation (`validateJQL`): max 5000 characters; `;`, `--`, `/*`, `\\` and control characters are rejected. Reduce the number of ids per batch or remove the offending characters.

### Jira search returns HTTP 410
Atlassian has retired `POST /rest/api/3/search` on some sites. The app automatically retries with `POST /rest/api/3/search/jql`. If you see repeated 410s in logs with no fallback, you are running a build older than the fallback (upgrade).

---

## Export Issues

### Export button does nothing / file empty
- Exports only contain the currently filtered dataset; apply filters and ensure the table shows rows.
- Browser pop-up/download blockers can stop the Blob download; allow downloads for `*.atlassian.net`.
- Only **CSV** and **Excel (.xlsx)** are supported. There is no PDF or JSON export.

### Spreadsheet shows a leading apostrophe in some cells
Intentional: values starting with `=`, `+`, `-` or `@` are prefixed to prevent CSV-injection.

---

## Performance Issues

### Dashboard slow on large projects
- The app stores evaluations as whole JSON arrays per key. Load time grows linearly with evaluation count, and Forge's per-value size limit is reached after a few hundred evaluations in one key (the global `ticket-evaluations` key first). Symptoms of hitting the limit: **Save Evaluation** fails with a storage error in `forge logs` while reads still work. See [Entity Storage Migration Plan](./Entity-Storage-Migration-Plan.md).
- Jira metadata is fetched in batches of 50 ids, at most 8 concurrent calls. A 2 000-ticket range needs ~40 calls.
- Advanced Analytics computes in a Web Worker and serves the last result from IndexedDB while refreshing in the background; the first load after a cache miss is the slow one.

### "429 Too Many Requests" in logs
The app has no server-side rate limiter; these come from Jira/Forge limits. Reduce concurrency by narrowing date ranges. The client retries up to 3 times with linear back-off before surfacing the error.

### Browser tab memory grows
Large exports build the whole workbook in memory. Export smaller date ranges.

---

## Build and Deployment (developers)

### `vite:worker-import-meta-url Invalid value 'iife' for option 'worker-format'`
Keep `worker: { format: 'es' }` in `vite.config.ts` and create workers via `new URL(..., import.meta.url)`.

### Deployment fails: scopes changed
After editing `permissions.scopes` in `manifest.yml`, every site needs `forge install --upgrade -e <env>`; Marketplace versions need re-approval.

### Resolver changes do not take effect
The manifest handler is `index.handler` → `src/index.js`. The root `index.js` file is a legacy copy and is **not** deployed; edit `src/index.js`.

### "Resolver not found" / `invoke` rejects immediately
Check the name against the list in [API Reference](./API-Reference.md). `makeApiRequest`, `debugUserFetch`, `testUserAPI` and `runPersonalDataReporter` were removed.

### Checking the scheduled job
```bash
forge logs -e production --function personalDataReporter --since 7d
```
Look for `[personalDataReporting]` summary lines (`closed`, `updated` counts) and `erasure failed` warnings.

---

## Diagnostics Cheat-Sheet

```bash
forge install list                       # where is the app installed
forge logs -e production --since 1h      # backend errors
forge lint                               # manifest/runtime problems
```

In Jira:
- **Apps → Manage apps → Quality Scorecard for Jira** – version, scopes, enable/disable.
- **Settings → Project Access & Recent Changes** – who changed the allow-list.
- **Settings → Roles & Access → audit list** – role/assignment/deletion history.

Audit entries written by the backend (`audit-log:*`) are not visible in the UI; include the approximate time and the user's account id when contacting support so they can be retrieved.

---

## Support Escalation

1. This guide and the [FAQ](../User-Guide/FAQ.md).
2. **support@nexusfrontier.net** — see [Support SLA](../Legal/Support-SLA.md) for response targets. Include: exact error text, steps to reproduce, project key, approximate time (with timezone), affected user's account id, browser, and screenshots.
3. Atlassian Support for platform-level outages: https://status.atlassian.com and https://support.atlassian.com.

## Additional Resources

- [Architecture Overview](./Architecture.md)
- [API Reference](./API-Reference.md)
- [Installation Guide](./Installation.md)
- [Development Guide](./Development.md)
- [Security Documentation](./Security.md)

---

*Last Updated: October 2026 — reflects app version 1.3.0*
