# Documentation Audit Report — October 2026

**Scope:** Every document in this repository (`Technical/`, `User-Guide/`, `Legal/`, GitHub Pages wrappers, `README.md`, `_config.yml`) was verified line-by-line against the application source in the `Quality Compass App` repository (working tree as of October 25, 2026; `package.json` version 1.3.0; production Forge deployment 7.x; `manifest.yml` runtime `nodejs22.x`).

**Outcome:** Every document was found to describe an earlier or planned version of the product rather than the shipped one. All documents have been rewritten or amended. Changes are left **uncommitted** for review.

**Why the repository looked stale:** the last commit in this repository is `71043f8` (June 2, 2026, app icon). The last commit in the app repository is `7bb2649` (June 2, 2026). The August–September 2026 security hardening (server-side permission enforcement, storage key allow-lists, scope reduction, removal of debug resolvers) that is running in production **exists only as uncommitted changes in the app repository working tree** (21 modified/untracked paths at the time of this audit). Nothing after June 2 was ever committed, so neither repository could reflect it.

---

## 1. Summary of findings by area

| Area | Documents | State before audit | Main problems |
|------|-----------|--------------------|---------------|
| Technical | 7 | Written from an early design (v1.0.0), never updated | Non-existent resolvers (`makeApiRequest`), removed scope `write:jira-work`, in-memory/per-user rate limiter that does not exist, retention jobs that never run, "Edit Issues" Jira permission gating that is not how authorization works, wrong Node runtime, fictional alerting/monitoring configuration |
| User Guide | 4 | Describes generic/aspirational features | Automation, alerts, email notifications, keyboard shortcuts, Help menu, PDF export, in-place evaluation editing — none exist; tab names, filters and card titles did not match the UI |
| Legal | 5 | v1.0–1.3 (March–May 2026) | Scope list includes `write:jira-work`; consent records and rate-limit data listed as stored personal data; 365/90/730/180/7-day retention periods that are not enforced; "built-in" self-service GDPR export/erasure that does not exist; group-membership admin check (actual check is the Jira `ADMINISTER` permission); PDF export; ToS response times contradict the Support SLA |
| GitHub Pages wrappers | 5 | Abridged v1.0 copies (March 7, 2026) | Out of sync with `Legal/*` for five months; same stale claims as above |
| `_config.yml` / `README.md` | 2 | March 2026 | `baseurl` pointed at a different repository name; README version block 1.0.0 / 1.1.0 |

---

## 2. Discrepancies found (docs vs. code)

Line numbers refer to the **committed (HEAD) version** of each document before this audit; code references are to the app repository.

### 2.1 Technical documentation

| # | Document (HEAD line) | Claim | Reality in code | Fix applied |
|---|----------------------|-------|-----------------|-------------|
| T1 | `Technical/Architecture.md:97`, `API-Reference.md:32–47` | `makeApiRequest` resolver is the generic Jira proxy | Removed during hardening; no generic proxy exists. Also removed: `debugUserFetch`, `testUserAPI`, `runPersonalDataReporter` | Resolver list rebuilt from `src/index.js` (35 resolvers); "Removed Resolvers" section added to API Reference |
| T2 | `Architecture.md:271`, `Installation.md:233`, `Security.md:82`, `Troubleshooting-Technical.md:303`, `Entity-Storage-Migration-Plan.md:276,287` | Scopes include `write:jira-work` | `manifest.yml:43–47`: `read:jira-work`, `storage:app`, `read:jira-user`, `report:personal-data` | Scope lists corrected everywhere |
| T3 | `Architecture.md:64`, `Security.md:206–245`, `API-Reference.md:390,440`, `Installation.md:133` | In-memory / per-user, per-endpoint rate limiter; `RATE_LIMIT` error code | `src/middleware/rateLimiter.js` exists but is not imported anywhere. Protection = Atlassian platform limits + client `invokeLimited` (max 8 concurrent) + `withRetry` (3 attempts) + `dedupeRequest` | Rewritten as platform/client/application layers; dead module listed |
| T4 | `Security.md:172`, `Security.md:340` | 90-day audit retention enforced; self-service `exportUserData` available | No automatic retention job runs and there is no self-service export | Documented as "planned, not enforced"; retention described as event-driven |
| T5 | `Installation.md:96`, `Security.md:68` | Evaluation requires Jira "Edit Issues"; role matrix View/Create/Edit/Admin | Authorization is the app's own permission set (`DEFAULT_PERMISSIONS`: `dashboard.access`, `tab.*`, `ticket.evaluate`, `evaluation.delete`) resolved by `computeEffectivePermissions`; admin = Jira `ADMINISTER` via `/rest/api/3/mypermissions` or optional `settings-admin-group` | Authorization model section rewritten; permission table replaced |
| T6 | `Security.md:380,458`, `Troubleshooting-Technical.md:684–688`, `Installation.md:353` | Automated security alerts, `ALERT_THRESHOLDS` configuration | No alerting exists; `security-alert:<ts>` keys are written but nothing consumes them | Removed; diagnostics section describes `forge logs` and audit keys |
| T7 | `Troubleshooting-Technical.md:249` and throughout | Generic error messages | Actual strings: `Storage key not permitted for <op>`, `Not allowed` toast, `ACCESS_DENIED` audit event, 410 fallback to `POST /search/jql` | Troubleshooting rebuilt around real strings |
| T8 | `Development.md` (whole) | Node 18/20, test suite, generic Forge workflow | `manifest.yml:37` `nodejs22.x`; no `test` script in `package.json`; `security-check` = `npm audit --audit-level=high`; tunnel on 127.0.0.1:8080 | Rewritten; manual test checklist added; "never `forge deploy -e production` without explicit approval" rule carried over from `AGENTS.md`/handoff |
| T9 | `Architecture.md` storage section | No storage size limit mentioned | Evaluations stored as arrays per key (~400–700 B each) → Forge value limit reached at a few hundred evaluations per key | Storage limit note added to Architecture and Troubleshooting; migration plan marked **Planned — not implemented** |
| T10 | `Architecture.md` component list | Lists components as if all active | `EvaluatorNameManager.tsx`, `SmartEvaluatorSync.tsx`, `components/admin/analytics/{OverviewTab,AssigneeTab,CriteriaTab,DataTab}.tsx`, `pages/Index.tsx`, `middleware/rateLimiter.js`, `utils/errorHandler.js`, root `index.js` are unused | Dead-code note added |

### 2.2 User Guide

| # | Document (HEAD line) | Claim | Reality | Fix applied |
|---|----------------------|-------|---------|-------------|
| U1 | `Workflows.md:436–495` | Settings → Automation; alert rules; email/in-app notifications | None exist | Section removed; "Not included" list added to Feature Guide |
| U2 | `Getting-Started.md:184` | Keyboard shortcuts; Help menu | None exist | Removed |
| U3 | `FAQ.md:25–29`, `Feature-Guide.md:192` | Evaluations can be edited in place | No edit; a new evaluation supersedes (form pre-fills from latest); deletion requires `evaluation.delete` | Corrected |
| U4 | `FAQ.md:180`, `Feature-Guide.md:273` | "Edit Issues" Jira permission needed to evaluate | `ticket.evaluate` app permission via Roles & Access | Corrected |
| U5 | All four | Tab/card/filter names generic | Actual: Analytics Dashboard / Agent Performance / Evaluator Analytics / Advanced Analytics / Metrics / Settings (sub-tabs "Project Access & Recent Changes", "Roles & Access", "Coverage Targets"); real card titles, filters, bands (<5 / 5–7.9 / ≥8), bias threshold ±2.5, Missing Evaluations = Done-status tickets by created date (cap 500), hidden tickets stored site-wide | Every screen described from the components |
| U6 | `Feature-Guide.md` export section | PDF / generic export | CSV and Excel only; fixed file names `analytics-dashboard|agent-performance|evaluator-analytics|advanced-analytics-<project>-<date>`; sheet names documented | Corrected |
| U7 | `FAQ.md` | Linked `Technical/README.md` | File does not exist | Link fixed |
| U8 | `Getting-Started.md`, `FAQ.md`, `Feature-Guide.md` (first draft of this audit) | "View History shows every evaluation" | `getEvaluationHistory` defaults to page 1 / 10 items and the dialog never pages (`evaluationService.ts:127–140`, `EvaluationHistory.tsx:53`) | Corrected to "10 most recent" |
| U9 | `Feature-Guide.md` Advanced Analytics | Date filter described like the other tabs | Advanced Analytics filters by **evaluation timestamp** (`advancedAnalyticsService.ts:969–984`); others by ticket created date; coverage by resolution date. Export-format control shows *Excel* first while state defaults to `csv` (`AdvancedAnalyticsDashboard.tsx:28,173`) | Documented; user told to pick the format explicitly |
| U10 | `Feature-Guide.md` Settings | "Add projects" described inside the project dashboard | `AdminView.tsx:241–244` renders `ProjectAccessCard` with `restrictToCurrentProject`, so adding is only possible on the landing page; Recent Changes shows 10 entries (`ProjectAccessCard.tsx:441`) | Corrected |
| U11 | `Feature-Guide.md` Metrics | **Required** switch implied enforcement | `EvaluationForm.tsx` never reads `required` | Marked informational |
| U12 | `Feature-Guide.md` Export | — | Advanced export *Patterns* sheet is always "N/A" (`getPeakPerformancePatterns` no longer called, `advancedAnalyticsService.ts:384–385,1049–1058`); Agent export "Team Average Score" is unweighted vs weighted on screen (`exportService.ts:251,288`) | Documented |

### 2.3 Legal documentation

| # | Document (HEAD line) | Claim | Reality | Fix applied |
|---|----------------------|-------|---------|-------------|
| L1 | `Privacy-Policy.md:243,306`; `Cloud-Security-Statement.md:76,113,352`; `DPA.md:337` | `write:jira-work` requested ("reserved for future use") | Not in `manifest.yml` | Removed; note that earlier versions requested it |
| L2 | `Privacy-Policy.md:56–57`; `CSS.md:155,243,321`; `DPA.md:78`; `cloud-security.md:67`; `privacy.md:58` | GDPR consent records and rate-limit data stored | `recordUserConsent` never called; no rate-limit storage | Rows removed; display-name caches and personal-data index added |
| L3 | `Privacy-Policy.md:148–150`; `CSS.md:143–145,247,345`; `DPA.md:236–238,332`; wrappers | Fixed retention: 365 / 90 / 730 / 180 / 7 days | No automatic expiry runs | Rewritten as event-driven retention tables (user deletion, account closure, provider erasure, uninstall) |
| L4 | `Privacy-Policy.md:168,183,191`; `DPA.md` §5.5, §10.2; `ToS.md:180`; `CSS.md:321` | Built-in self-service export/erasure; account closure erases *all* personal data | Automatic erasure on account closure covers the site-wide evaluation list, the name override and the index entry; complete erasure of residual references is performed by the provider on request (disclosed as a known limitation) | Erasure scope described honestly; admin/processor-assisted 30-day path documented; no self-service |
| L5 | `Privacy-Policy.md:38,308`; `CSS.md:102,115`; `DPA.md:74,309`; `cloud-security.md:44` | Admin verified by Jira group membership | Jira global `ADMINISTER` permission check; group only if `settings-admin-group` configured (no UI) | Corrected |
| L6 | `Privacy-Policy.md:241`; `CSS.md:215–221,242`; `DPA.md:114,290`; `ToS.md:78,145`; `privacy.md:147`; `dpa.md:52` | Per-user, per-endpoint rate limiting | See T3 | Replaced with platform/client limits and audit of denied access |
| L7 | `Terms-of-Service.md:176` | Export in Excel and **PDF** | CSV and Excel only | Corrected |
| L8 | `Terms-of-Service.md` §6.3 | Response targets 24 h / 48 h / 5 bd / 10 bd | `Support-SLA.md` says 4 / 8 / 24 / 48 business hours | ToS now references the SLA and reproduces its table; SLA prevails on conflict |
| L9 | `Privacy-Policy.md:84` | Preferences stored on the basis of consent | No consent is collected; preferences are set by the user's own action | Legal basis table corrected (legitimate interest / contract) |
| L10 | `Cloud-Security-Statement.md` §10 | Implies independent assessment | No independent penetration test has been performed; Ecoscanner + `npm audit` + SBOM | "Independent Assessment" row added stating this |

### 2.4 Repository / publishing

| # | File | Problem | Fix applied |
|---|------|---------|-------------|
| R1 | `_config.yml` | `baseurl: "/NexusFrontier-Documentation"` while this repository is `QualityCompass-Documentation`, so project Pages assets/links would resolve to the wrong path | Set to `/QualityCompass-Documentation` with a comment |
| R2 | `privacy.md`, `terms.md`, `dpa.md`, `cloud-security.md`, `sla.md` | Abridged v1.0 copies, out of sync with `Legal/*` since March | Regenerated as full copies of the `Legal/*` sources with Jekyll front matter and Pages-friendly links (`./dpa`, `./privacy`, …) |
| R3 | `README.md` | App 1.0.0 / Docs 1.1.0 / March 2026 | Version block updated; links to migration plan and this report added; note that Pages wrappers are generated from `Legal/` |

---

## 3. Documents changed

| Document | Change | New version / footer |
|----------|--------|----------------------|
| `Technical/Architecture.md` | Rewritten | "Last Updated: October 2026 — reflects app version 1.3.0 (production deployment 7.x)" |
| `Technical/API-Reference.md` | Rewritten (all 35 resolvers, payload/response/authorization, validators, removed resolvers) | October 2026 |
| `Technical/Security.md` | Rewritten | October 2026 |
| `Technical/Installation.md` | Rewritten (Marketplace install, first-run, developer deploy, uninstall) | October 2026 |
| `Technical/Development.md` | Rewritten | October 2026 |
| `Technical/Troubleshooting-Technical.md` | Rewritten | October 2026 |
| `Technical/Entity-Storage-Migration-Plan.md` | Status header amended: **Planned — not yet implemented** | — |
| `User-Guide/Getting-Started.md` | Rewritten | October 2026 |
| `User-Guide/Feature-Guide.md` | Rewritten (+ "Not included" section) | October 2026 |
| `User-Guide/Workflows.md` | Rewritten | October 2026 |
| `User-Guide/FAQ.md` | Rewritten | October 2026 |
| `Legal/Privacy-Policy.md` | Sections 2, 3, 5, 7, 8, 10, 16 amended; §17 change history added | **v1.4**, October 25, 2026 |
| `Legal/Terms-of-Service.md` | §5.2, §6.3, §9.3, §9.4 amended; change history added | **v1.2**, October 25, 2026 |
| `Legal/Data-Processing-Agreement.md` | §4.3, §5.3, §5.5, §10.1, §10.2, Annex A.2/A.5/A.9/A.10 amended; change history added | **v1.3**, October 25, 2026 |
| `Legal/Cloud-Security-Statement.md` | §2–§12 amended | **v1.4**, October 25, 2026 |
| `Legal/Support-SLA.md` | No change (content verified accurate) | March 15, 2026 |
| `privacy.md`, `terms.md`, `dpa.md`, `cloud-security.md`, `sla.md` | Regenerated from `Legal/*` | match sources |
| `_config.yml`, `README.md` | Amended | — |
| `DOCUMENTATION-AUDIT-REPORT.md` | New (this file) | — |

The `Quality Compass App/Documentation/Legal/*` copies in the app repository differ from the pre-audit public copies only in version/date lines; they should be replaced with the new `Legal/*` files when the app repository is next committed.

---

## 4. Code observations

Verifying the documents also surfaced a number of implementation observations (unused modules, UI labels that do not match behaviour, items already disclosed as known limitations in the Technical and Legal documents, and internal documentation drift in the application repository). These are tracked internally in the application repository (Documentation/Security/Code-Findings-Documentation-Audit-2026-10.md) and are not reproduced here.

---
## 5. Verification method

1. Read every document in this repository and the app-only docs (`Documentation/Personal-Data-Reporting`, security runbooks, Marketplace listing, `AGENTS.md`, `MARKETPLACE-SUBMISSION-HANDOFF.md`).
2. Read the backend (`src/index.js`, `src/services/*`, `src/utils/*`, `src/personalDataReporting.js`, `manifest.yml`, `package.json`) and the frontend (`App.tsx`, every view/tab component under `src/components`, `src/services/*`, `vite.config.ts`).
3. For each documented claim, located the implementing code or confirmed its absence; corrected the document to match.
4. Checked internal links and anchors across the updated set.

Nothing was committed, pushed or deployed as part of this audit.
