# Development Guide

## Related Documentation
- [Architecture Overview](./Architecture.md) - System architecture and design
- [API Reference](./API-Reference.md) - Resolver documentation
- [Installation Guide](./Installation.md) - Deployment steps
- [Security Documentation](./Security.md) - Security rules every change must respect
- [Technical Troubleshooting](./Troubleshooting-Technical.md) - Debugging and issues

## Prerequisites

```bash
node --version   # v22.x (matches Forge runtime nodejs22.x)
npm --version    # v10.x
forge --version  # latest @forge/cli
```

An Atlassian developer account and a Jira Cloud development site (free at developer.atlassian.com) are required.

## Initial Setup

```bash
git clone <repository-url>
cd "Quality Compass App"
npm ci
npm install -g @forge/cli
forge login
```

If you are working on your **own copy** of the app (not the published one), run `forge register` once to get a new app id in `manifest.yml`. Never run `forge register` against the shared repository checkout that targets the published app.

There is no `.env` file. Configuration lives in `manifest.yml` (modules, scopes, runtime) and in Forge storage (runtime settings).

## Project Structure

```
Quality Compass App/
├── manifest.yml                     # Forge modules, functions, scopes, runtime
├── package.json                     # scripts + dependencies (version 1.3.0)
├── vite.config.ts                   # Vite build (base './', worker.format 'es', manual chunks)
├── index.html                       # Custom UI entry
├── src/
│   ├── index.js                     # Forge resolvers (backend) – CommonJS
│   ├── personalDataReporting.js     # Weekly scheduledTrigger handler
│   ├── main.tsx / App.tsx           # React entry; App routes by Forge module key
│   ├── pages/                       # ProjectLandingPage, AdminView, IssuePanelView
│   ├── components/                  # admin/, evaluation/, shared/, ui/ (shadcn)
│   ├── contexts/                    # JiraContext, ProjectContext, ThemeContext
│   ├── hooks/                       # usePrefetch, useTicketDetailSheet, use-toast, …
│   ├── services/                    # Frontend services (.ts) + backend helpers (.js)
│   │   ├── auditService.js          # backend
│   │   ├── gdprService.js           # backend
│   │   ├── personalDataIndex.js     # backend
│   │   ├── workers/advancedAnalyticsWorker.ts
│   │   └── utils/api.ts             # invoke wrapper (limiter, dedupe, retry)
│   ├── validators/inputValidator.js # backend input validation
│   ├── types/                       # TS types (index.ts, forge.d.ts, jira.d.ts)
│   ├── middleware/rateLimiter.js    # legacy – not imported anywhere
│   └── utils/errorHandler.js        # legacy – not imported anywhere
├── dist/                            # Vite output; referenced by manifest resources
├── Documentation/                   # developer copy of the public docs
└── index.js                         # legacy root copy – NOT used (handler is src/index.js)
```

Backend code (`src/index.js`, `src/personalDataReporting.js`, `src/services/*.js`, `src/validators`) runs in the Forge Node.js runtime and uses `require`. Frontend code (`.ts/.tsx`) is bundled by Vite and runs in the Custom UI iframe. Do not import backend `.js` modules from frontend code or vice versa.

## npm Scripts

| Script | Purpose |
|--------|---------|
| `npm run dev` | Vite dev server on `127.0.0.1:8080` (frontend only, uses mock data where the bridge is unavailable) |
| `npm run build` | `vite build` → `dist/` (the `build:resolver` step is a no-op; resolvers are deployed from `src/`) |
| `npm run build:dev` | Development-mode build |
| `npm run lint` | ESLint (flat config, TypeScript + React hooks rules) |
| `npm run security-check` | `npm audit --audit-level=high` |
| `npm run preview` | Preview the production build locally |

There is currently **no automated test suite** (`npm test` is not defined). Verification is manual in a development site; see "Manual test checklist" below.

## Development Workflow

### 1. Deploy to the development environment
```bash
npm run build
forge deploy -e development
forge install -e development --site <your-dev-site>.atlassian.net --product jira   # first time
forge install --upgrade -e development                                           # after scope/module changes
```

### 2. Iterate with the tunnel
```bash
forge tunnel -e development
```
With the tunnel running, resolver changes in `src/` take effect without redeploying. For frontend changes rebuild with `npm run build` (the tunnel serves `dist/`), or use `npm run dev` for pure UI work.

### 3. Logs
```bash
forge logs -e development
forge logs -e production --since 1h
```

### Environments
`development`, `staging` and `production` exist. Production is the Marketplace-listed app; **never** run `forge deploy -e production` without explicit approval and never while an Atlassian Marketplace review is open.

## Adding a Resolver

```javascript
// src/index.js
resolver.define('myNewResolver', async ({ context, payload }) => {
  const { projectKey } = payload || {};

  // 1. Validate inputs
  const validatedProjectKey = validateProjectKey(projectKey);

  // 2. Enforce project allow-list and permission – server-side, always
  await ensureProjectAccessConfigured(validatedProjectKey);
  const perms = await computeEffectivePermissions(context, validatedProjectKey);
  if (!perms.has('tab.settings')) {
    await logAuditEvent(AUDIT_EVENTS.ACCESS_DENIED, { accountId: context.accountId, projectKey: validatedProjectKey }, { action: 'my_resolver_denied' });
    throw new Error('Not allowed');
  }

  // 3. Do the work
  const data = await storage.get(`my-key:${validatedProjectKey}`);
  return { success: true, data };
});
```

Rules:
- Do **not** extend `GENERIC_READ_KEYS` / `GENERIC_WRITE_KEYS` / `GENERIC_DELETE_KEYS` for privileged data; add a dedicated resolver with its own permission check instead.
- Do **not** reintroduce a generic Jira proxy resolver.
- Any new storage key that holds account ids must be added to the discovery scan in `personalDataIndex.js` so it is covered by Personal Data Reporting, and to the erasure path in `gdprService.deleteUserData`.
- Document the resolver in [API-Reference.md](./API-Reference.md).

### Calling it from the frontend
```typescript
import { invokeLimited, withRetry } from '@/services/utils/api';   // limiter + dedupe (+ optional retry)

const res = await withRetry(() =>
  invokeLimited<{ success: boolean; data: unknown }>('myNewResolver', { projectKey })
);
```

## Frontend Notes

- **Styling**: Tailwind + shadcn/ui components in `src/components/ui`. Avoid inline styles (CSP).
- **Charts**: Recharts. Keep heavy transforms in services, not components.
- **Caching**: use `cacheService` (`CACHE_TTL.SHORT|MEDIUM|LONG|VERY_LONG`) and, for expensive analytics, `persistedCache` with a stale-while-revalidate pattern as in `advancedAnalyticsService.ts`.
- **Workers**: create with
  ```ts
  new Worker(new URL('./workers/advancedAnalyticsWorker.ts', import.meta.url), { type: 'module' })
  ```
  `vite.config.ts` already sets `worker: { format: 'es' }`. Do not use the `?worker` import suffix.
- **Dates**: all JQL date filters use an exclusive end bound (`< endExclusive`) – see `ticketService.ts`. Preserve this when adding filters.
- **Jira links**: build with `services/utils/jiraUrl.ts` (uses `getJiraBaseUrl`).
- **Permissions in UI**: gate tabs/buttons with `AccessControlService.canAccess()` / `getMyPermissions()`, but remember the backend is the real gate.

## Manual Test Checklist

Run in a development site before any deploy:

- [ ] Issue panel loads for an allowed project; sliders show current criteria; **Save Evaluation** succeeds; **View History** lists it.
- [ ] Issue panel for a user without `ticket.evaluate` cannot save (error toast, backend rejects).
- [ ] Project landing page: admin sees Configure Projects when none allowed; non-admin sees the "ask your administrator" message.
- [ ] Analytics Dashboard: date range filter, all cards, raw table pagination/sorting, CSV and Excel export.
- [ ] Agent Performance: search, sort, threshold filter, detail modal, export.
- [ ] Evaluator Analytics: bias flagging, detail sheet, Missing Evaluations load/remove, delete evaluations (with/without `evaluation.delete`).
- [ ] Advanced Analytics: Coverage/Trends/Complexity/Distribution tabs; Refresh re-computes; persisted cache serves instantly on reload.
- [ ] Metrics: add/edit/reorder/remove criteria as admin; non-admin edit is rejected by backend.
- [ ] Settings: project add/remove with Recent Changes; role create/edit/delete; user add/remove; audit log loads; coverage targets save.
- [ ] `forge logs` shows no unhandled errors.
- [ ] `npm run lint` and `npm run security-check` pass.

## Code Standards

- TypeScript for all frontend code; functional components with hooks.
- Files: components `PascalCase.tsx`, services/hooks `camelCase.ts`.
- Constants `UPPER_SNAKE_CASE`; types/interfaces `PascalCase`.
- Use `logger` (`services/utils/logger.ts`) instead of `console.*` in frontend code; backend code may use `console.error` for Forge logs but must never log personal data or tokens.
- Conventional commits: `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`, `security:`.

## Security Practices

- Validate every payload field with `inputValidator.js` helpers.
- Check permissions server-side in every resolver that reads or writes non-public data.
- Never return Jira e-mail addresses to the frontend.
- Escape user-supplied text in JQL (quote and escape `"`), and never interpolate unvalidated project keys.
- Keep `manifest.yml` scopes minimal; a scope change requires `forge install --upgrade` on every site and a Marketplace re-review.
- Run `npm run security-check` and regenerate `sbom.json` (`npx @cyclonedx/cyclonedx-npm --output-file sbom.json --output-format JSON`) before a release.

## Release Checklist

- [ ] Version bumped in `package.json`
- [ ] `npm run lint`, `npm run security-check` clean
- [ ] Manual test checklist complete in development
- [ ] `forge deploy -e staging` and smoke test
- [ ] Documentation updated (this repo + `Documentation/` copy in the app repo)
- [ ] Marketplace listing/version notes prepared
- [ ] Explicit approval obtained for `forge deploy -e production`
- [ ] All changes committed and pushed — production must never run code that is not in git

## Resources

- [Forge Documentation](https://developer.atlassian.com/platform/forge/)
- [Forge Custom UI](https://developer.atlassian.com/platform/forge/custom-ui/)
- [Personal Data Reporting API](https://developer.atlassian.com/platform/forge/apis-reference/privacy-api/)
- [Jira Cloud REST API v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/)
- [Vite](https://vitejs.dev/) · [React](https://react.dev/) · [Recharts](https://recharts.org/)

---

*Last Updated: October 2026 — reflects app version 1.3.0*
