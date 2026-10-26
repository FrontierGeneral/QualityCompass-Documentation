---
layout: page
title: Cloud Security
permalink: /cloud-security/
---

# Quality Scorecard for Jira — Cloud Security Statement

**Effective Date:** March 7, 2026
**Last Updated:** October 25, 2026
**Version:** 1.4

---

## 1. Overview

This Cloud Security Statement describes the security practices, architecture, and controls implemented for the Quality Scorecard for Jira application ("the App"), a Jira Cloud application built on the Atlassian Forge platform.

Quality Scorecard for Jira is designed with a security-first approach, leveraging Atlassian's enterprise-grade infrastructure for all data storage, authentication, and runtime execution.

---

## 2. Architecture and Hosting

### 2.1 Platform

Quality Scorecard for Jira is built entirely on the **Atlassian Forge** platform, Atlassian's serverless, cloud-native app development framework. This means:

- **No self-hosted infrastructure:** The App runs on Atlassian's managed infrastructure (AWS Lambda). Nexus Frontier does not operate, manage, or have access to any servers.
- **No external services:** The App does not communicate with any servers, APIs, or services outside of Atlassian's infrastructure.
- **Sandboxed execution:** Forge Functions execute in an isolated, sandboxed environment managed by Atlassian.
- **Managed runtime:** Node.js 22.x, managed and patched by Atlassian.

### 2.2 Data Storage

All persistent data is stored in **Atlassian Forge Key-Value Storage**, which provides:

| Security Property | Implementation |
|-------------------|---------------|
| Encryption at Rest | AES-256 (Atlassian-managed) |
| Encryption in Transit | TLS 1.2+ |
| Tenant Isolation | Data isolated per app per Jira Cloud site |
| Access Control | Accessible only through Forge runtime APIs |
| Physical Security | Atlassian data centers (SOC 2, ISO 27001, ISO 27018 certified) |
| Backup and Recovery | Managed by Atlassian |

### 2.3 Network Architecture

```
┌──────────────────────────────────────────────────────────┐
│                  Atlassian Cloud                          │
│                                                          │
│  ┌──────────────┐    ┌────────────────────────────────┐  │
│  │  Jira Cloud   │◄──►│  Quality Scorecard for Jira (Forge App)   │  │
│  │  REST APIs    │    │  - Forge Functions (resolvers)  │  │
│  └──────────────┘    │  - Forge Key-Value Storage      │  │
│                       └────────────┬───────────────────┘  │
│                                    │                      │
│                                    ▼                      │
│                       ┌────────────────────────────────┐  │
│                       │  Custom UI (sandboxed iframe)   │  │
│                       │  - React frontend               │  │
│                       │  - Browser-local caches only    │  │
│                       └────────────────────────────────┘  │
│                                                          │
│           ↓ Personal Data Reporting (weekly)              │
│   ┌─────────────────────────────────────────────────┐    │
│   │  api.atlassian.com/app/report-accounts/         │    │
│   │  via privacy.reportPersonalData (Atlassian API) │    │
│   └─────────────────────────────────────────────────┘    │
│                                                          │
│      NO THIRD-PARTY CONNECTIONS OUTSIDE ATLASSIAN          │
└──────────────────────────────────────────────────────────┘
```

**Key point:** The App makes no connections to any third-party domain. The only outbound network call leaving the Forge function runtime is the weekly Personal Data Reporting call to `api.atlassian.com` (Atlassian's own API). That call is performed via the `privacy.reportPersonalData` helper in `@forge/api`, which handles app authentication, batching, and the HTTP request internally — so no `remotes:` block or `permissions.external.fetch` declaration is required in `manifest.yml`. The `manifest.yml` declares only the minimum API scopes needed:

```yaml
permissions:
  scopes:
    - read:jira-work
    - storage:app
    - read:jira-user
    - report:personal-data
```

---

## 3. Authentication and Authorization

### 3.1 Authentication

Authentication is handled entirely by the Atlassian Forge platform:

- Users authenticate through their organization's Atlassian account (Atlassian SSO)
- The App does not collect, store, or manage any user credentials
- The App does not use Personal Access Tokens (PATs), API tokens, or shared secrets
- Each Forge Function invocation receives the authenticated user's context automatically

### 3.2 Authorization

The App implements a multi-layer authorization model:

| Layer | Mechanism | Description |
|-------|-----------|-------------|
| **Platform** | Atlassian Forge | Only authenticated Jira users can access the App |
| **Admin Verification** | Jira global permission check | Administrative functions verify that the user holds Jira's global **Administer Jira** permission via `GET /rest/api/3/mypermissions?permissions=ADMINISTER`. Optionally, a Customer may designate a Jira group whose members are also treated as App administrators. |
| **RBAC** | App-Level Roles | Per-project role-based access control with nine permissions (dashboard access, five dashboard tabs, settings, evaluate tickets, delete evaluations); roles and assignments are stored per project and managed only by administrators |
| **Server-Side Enforcement** | Forge resolver checks | Every backend function re-checks the caller's effective permissions; generic storage access is limited to an explicit allow-list of keys, and all Jira searches are constrained to the administrator-approved project list |
| **Data Scoping** | Project-Level Isolation | Users can only open dashboards, evaluate tickets and fetch ticket metadata for projects they have been granted access to; ticket metadata lookups additionally verify the user's Jira **Browse Projects** permission |

### 3.3 API Scopes (Least Privilege)

The App requests only the minimum Jira API scopes necessary. It does not request any write scope and never modifies Jira issues.

| Scope | Justification |
|-------|---------------|
| `read:jira-work` | Read issue, project and permission data (key, summary, status, priority, type, assignee, reporter, dates) for evaluation context, analytics and access checks |
| `storage:app` | Store and retrieve evaluations, criteria, roles, audit logs and configuration in Forge Storage |
| `read:jira-user` | Read user profiles (Account ID, Display Name; group membership only when an optional admin group is configured) for identification and admin verification |
| `report:personal-data` | Required by Atlassian's Personal Data Reporting API. Allows the App to report to Atlassian, on a weekly schedule, the list of Account IDs for which it stores personal data so Atlassian can notify the App when accounts are closed or updated (see §4.5). |

---

## 4. Data Protection

### 4.1 Data Classification

| Classification | Data Types | Protection Level |
|---------------|------------|-----------------|
| **Confidential** | Jira Account IDs and Display Names stored in evaluations, role assignments, caches and audit logs | High — encrypted at rest and in transit, access controlled |
| **Internal** | Evaluation scores, free-text feedback, audit event details | Standard — encrypted at rest and in transit |
| **Configuration** | Evaluation criteria definitions, coverage targets, project allow-list | Standard — no personal data except the Account ID of the administrator who made a change |

E-mail addresses are not stored by the App. Issue content is read from Jira at display time and cached only in the user's browser.

### 4.2 Personal Data Handling

| Principle | Implementation |
|-----------|---------------|
| **Data Minimization** | Only data necessary for evaluation functionality is collected; e-mail addresses, avatars and issue descriptions are not stored |
| **Purpose Limitation** | Data is used solely for quality assurance evaluation purposes |
| **Storage Limitation** | Data is kept for the life of the installation and removed on account closure (Personal Data Reporting), on administrator deletion, or on uninstall; see §4.3 |
| **Integrity and Confidentiality** | Encryption at rest and in transit; input validation; server-side access controls |

### 4.3 Data Retention

The App does not apply automatic time-based expiry to stored data. Data is retained until one of the following occurs:

| Trigger | Effect |
|---------|--------|
| Administrator deletes a ticket's evaluations (requires "Delete Evaluations" permission) | Evaluations for that ticket are removed from site and project lists; the action is audited |
| Atlassian reports an account as closed (weekly Personal Data Reporting, §4.5) | The account's evaluations are removed from the site-wide list; its display-name override and personal-data index entry are deleted; the erasure is audited |
| Administrator removes a user's roles or a project | The relevant assignment / allow-list entry is removed |
| Customer requests erasure through Nexus Frontier | Complete removal of a user's data from all storage locations by the Processor |
| App uninstalled | All App data for the site is deleted by the Forge platform after the standard grace period (§11) |

Server-side caches of user profile data refresh after 1 hour (user info) and 24 hours (project user lists). Client-side caches expire within 1 minute to 2 hours.

A 90-day retention for backend audit entries is defined in the code base but not yet enforced automatically; audit entries currently persist until uninstall.

### 4.4 Data Subject Rights

Data subject rights are supported through administrator tooling in the App and Processor assistance; there is no end-user self-service export/erase function:

- **Access / Portability:** administrators export evaluation data (CSV/Excel) from the dashboards; role assignments and project access history are visible under Settings; complete compilations are provided by Nexus Frontier on request
- **Erasure:** automatic on account closure (scope described in §4.3); administrator-initiated per ticket; complete per-user erasure performed by Nexus Frontier on request
- **Rectification:** display names follow the Jira profile; scores are corrected by saving a superseding evaluation
- **Restriction / Objection:** administrators remove the user's role assignments

Details and timelines are in the [Privacy Policy](./privacy) §8 and the [Data Processing Agreement](./dpa).

### 4.5 Personal Data Reporting API

The App implements Atlassian's **Personal Data Reporting API** as required for Marketplace cloud apps that store user personal data ([Forge user privacy guide](https://developer.atlassian.com/platform/forge/user-privacy-guidelines/)). A scheduled trigger runs weekly and:

1. Compiles the list of Atlassian Account IDs for which the App stores personal data, along with the timestamp at which each user's data was last refreshed.
2. Calls `privacy.reportPersonalData()` from `@forge/api`, which sends the list to `https://api.atlassian.com/app/report-accounts/` in batches of up to 90 accounts. The platform helper handles app authentication, batching, and the HTTP call internally.
3. Processes Atlassian's response: accounts marked `closed` are erased automatically via the existing GDPR erasure flow; accounts marked `updated` are refreshed from the Atlassian user APIs.
4. Honors the Atlassian-provided `Cycle-Period` header and `Retry-After` rate-limit headers.

The reporting workflow is the only outbound network destination the App calls outside its own Forge function runtime, and it targets Atlassian's own API. The `report:personal-data` scope is declared in `manifest.yml`; no `remotes:` block or `permissions.external.fetch` entry is required because the privacy module handles the outbound call internally.

---

## 5. Application Security

### 5.1 Input Validation

All user-supplied input is validated and sanitized before processing:

| Input Type | Validation |
|------------|-----------|
| JQL Queries | Maximum length (5,000 characters) and rejection of statement separators, SQL/C-style comment markers, escaped backslashes and control characters; every search is additionally constrained to the administrator-approved project list |
| Project Keys | Pattern enforcement (`^[A-Z][A-Z0-9_-]{1,9}$`) |
| Issue Identifiers | Numeric id or `KEY-123` pattern |
| Account IDs | Pattern enforcement with length limit (255) |
| Pagination | Bounded `startAt` (≤ 100,000) and `maxResults` (≤ 100) |
| Evaluation Scores | Integer range 0–10 enforced by the UI control |
| Feedback Text | Rendered as text by React (no HTML interpretation); not stored in Jira |
| Storage Keys | Pattern enforcement (`^[a-zA-Z0-9:_-]{1,200}$`) plus an explicit allow-list of keys reachable from the client |

### 5.2 Common Vulnerability Protections

| Vulnerability | Mitigation |
|---------------|-----------|
| **Cross-Site Scripting (XSS)** | React output encoding, Content Security Policy (CSP) enforced by Forge, sandboxed iframe, no inline scripts or inlined assets in the build |
| **Cross-Site Request Forgery (CSRF)** | Forge platform handles CSRF protection natively |
| **Injection Attacks** | Input validation; Jira calls built with Forge's `route` template tag; project keys validated and quoted before use in JQL |
| **Broken Access Control** | Server-side permission checks in every resolver; storage-key allow-lists; removal of the former generic Jira proxy and debug functions |
| **CSV Injection** | Dangerous character prefixing in exported values |
| **Sensitive Data in Logs** | Forge runtime logs may include Atlassian Account IDs (opaque pseudonymous identifiers) for debugging and retry diagnostics. Display names, email addresses, evaluation content, comments, and other direct personal data are not logged. All logs are stored within Atlassian's hosted Forge logging infrastructure — not exported to any external service. |

### 5.3 Content Security Policy

The Forge platform enforces Content Security Policy (CSP) headers for the sandboxed Custom UI iframe, restricting script and style sources to the app's own resources.

### 5.4 Security Headers

Applied by the Forge platform:

| Header | Value |
|--------|-------|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` |
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `DENY` |
| `X-XSS-Protection` | `1; mode=block` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |

---

## 6. Rate Limiting and Abuse Prevention

### 6.1 Platform Controls

- Atlassian enforces Forge function invocation quotas and Jira REST API rate limits for every installation.
- Only authenticated Jira users of the Customer's site can invoke the App's functions.

### 6.2 Client-Side Controls

- Concurrency limiter (default: 8 in-flight requests) reduces contention
- In-flight request deduplication coalesces identical API calls
- Up to three attempts with linear back-off on failed calls

### 6.3 Application Controls

- Jira searches are capped at 100 results per request and are always constrained to approved projects
- Ticket metadata is fetched in batches of 50 identifiers
- The App does not implement its own per-user request counter; abuse protection relies on the platform controls above together with server-side authorization, and denied access attempts are recorded in the audit log

---

## 7. Audit and Monitoring

### 7.1 Audit Logging

Administrative and security-relevant events are logged:

| Event Category | Events recorded | Where visible |
|---------------|----------|----------|
| **Access Control** | Access denied (storage key outside allow-list, evaluation write without permission, evaluation delete without permission, reading assignments/audit log without permission) | Backend audit log |
| **Permission Changes** | Role created/updated/deleted, roles assigned (single/bulk), user access removed, evaluations deleted | Backend audit log **and** the Roles & Access audit list in Settings |
| **Project Access** | Project added/removed from the approved list, with actor and timestamp | Project Access & Recent Changes in Settings |
| **Personal Data** | User data erased following account closure | Backend audit log |

Each backend audit entry records a timestamp, event type, severity, the actor's Account ID, the project key (where applicable) and event-specific details. Evaluation creation is not audited separately because every evaluation record already carries its evaluator and timestamp and is preserved in the ticket's history.

### 7.2 Audit Log Retention

Backend audit entries are currently retained for the life of the installation (a 90-day automatic retention is planned). The in-App "Recent Changes" list shows the 20 most recent project-access changes.

### 7.3 Forge Platform Logs

The Atlassian Forge platform independently provides:
- Function invocation logs (accessible via `forge logs`)
- Error and exception tracking
- Performance metrics

---

## 8. Vulnerability Management

### 8.1 Dependency Management

- Dependencies are audited with `npm audit --audit-level=high` before every release and monthly; a CycloneDX SBOM (`sbom.json`) is regenerated for each release
- Security vulnerabilities are addressed per severity:
  - **Critical/High:** Patched within 14 days
  - **Medium:** Patched within 30 days
  - **Low:** Patched in the next scheduled release

### 8.2 Automated Scanning

- **Atlassian Ecoscanner:** Automatically scans all Marketplace app versions for vulnerabilities
- **Forge Lint:** Static analysis of manifest and code compliance (`forge lint`)
- **npm audit:** Dependency vulnerability scanning

### 8.3 Code Security Practices

- No hardcoded secrets, API keys, or credentials in source code
- No debug telemetry or test endpoints in production builds
- Security-focused code reviews for all changes
- Error messages do not expose internal system details

---

## 9. Incident Response

### 9.1 Procedure

1. **Detection:** Identification through audit logs, Forge platform monitoring, or user reports
2. **Assessment:** Severity classification and impact analysis
3. **Containment:** Isolate affected components; prevent further exposure
4. **Notification:** Notify affected Customers within 72 hours of confirmed Security Incidents involving personal data (per GDPR Article 33)
5. **Remediation:** Patch vulnerabilities, deploy fixes, restore services
6. **Post-Incident Review:** Document findings, update procedures, implement preventive measures

### 9.2 Contact

| | |
|---|---|
| **Security Contact** | Nexus Frontier Support |
| **Email** | support@nexusfrontier.net |
| **Subject Line** | Quality Scorecard for Jira — Security Incident |

---

## 10. Compliance

### 10.1 Atlassian Infrastructure Compliance

Quality Scorecard for Jira benefits from Atlassian's infrastructure certifications:

| Certification | Scope |
|--------------|-------|
| SOC 2 Type II | Security, Availability, Confidentiality |
| ISO 27001 | Information Security Management |
| ISO 27018 | Protection of PII in Public Clouds |
| CSA STAR | Cloud Security |

### 10.2 App-Level Compliance

| Requirement | Status |
|-------------|--------|
| GDPR Data Subject Rights | Supported via administrator export, automatic erasure on account closure, administrator deletion and Processor-assisted erasure (§4.4) |
| Personal Data Reporting | Implemented (weekly, `report:personal-data` scope) |
| Data Minimization | Only necessary data collected; no e-mail addresses stored |
| Least Privilege Scopes | Read-only Jira scopes plus app storage and personal-data reporting; no write scope |
| Encryption at Rest | AES-256 via Forge infrastructure |
| Encryption in Transit | TLS 1.2+ |
| No External Data Sharing | Zero third-party data transfers |
| Audit Logging | Access denials, permission changes, deletions and erasures logged |
| Input Validation | Backend validation of all resolver inputs |
| Secure Development Practices | Code review, dependency auditing, SBOM, Forge lint, Atlassian Ecoscanner |
| Independent Assessment | No third-party penetration test or certification of the App itself has been performed to date; platform certifications apply to the hosting layer |

---

## 11. Privacy & Security Tab Summary

This section maps directly to the Atlassian Marketplace Privacy & Security tab questions:

| Question | Response |
|----------|----------|
| Does the app store End-User Data outside Atlassian products? | **No.** All data is stored in Atlassian Forge Key-Value Storage. |
| Does the app process End-User Data outside Atlassian products or the user's browser? | **No.** All processing occurs within Atlassian Forge Functions or the user's browser. |
| Does the app log End-User Data? | **Yes.** Audit logs contain Account IDs and event details. Logs are stored in Forge Storage (within Atlassian products). |
| Does the app share End-User Data with third-party entities? | **No.** |
| Does the app support data residency? | Data residency is governed by Atlassian's Forge infrastructure. The App does not independently control data residency. |
| Does the app store End-User Data after a customer uninstalls the app? | **Yes — but only via the Atlassian Forge platform's standard 28-day soft-delete grace period.** The App itself does not retain or extend this period and maintains no independent data store. After 28 days, all data is permanently deleted by Atlassian. Minimum retention: 0 days. Maximum retention: 28 days. Reference: [Atlassian Forge Data lifecycle for hosted storage](https://developer.atlassian.com/platform/forge/storage-reference/hosted-storage-data-lifecycle/). |
| Does the app allow custom data retention periods? | **No.** The App does not apply time-based retention; data is retained for the life of the installation and removed on administrator deletion, account closure or uninstall (§4.3). |
| Is the provider a "data controller" under GDPR? | **No.** Nexus Frontier acts as a Data Processor. The Customer is the Data Controller. |
| Is the provider a "data processor" under GDPR? | **Yes.** With respect to all End-User Data processed through the App. |
| Does the app have a DPA? | **Yes.** Available at [Data Processing Agreement](./dpa). |
| Does the app transfer EEA data outside the EEA? | **No.** The App does not independently transfer data. Atlassian manages any cross-border transfers. |
| Does the app use full disk encryption at-rest for End-User Data stored outside of Atlassian or the user's browser? | **No.** Quality Scorecard for Jira does not store any End-User Data outside of Atlassian. All persistent storage is in Atlassian Forge Key-Value Storage, which is encrypted at-rest by Atlassian using AES-256. The Marketplace question applies only to data stored outside Atlassian — which is not the case for this app. |
| Does the app access PATs or shared secrets? | **No.** Authentication is handled natively by Forge. |
| API scopes requested | `read:jira-work`, `storage:app`, `read:jira-user`, `report:personal-data` |
| Scope justification | Read issue and user data for evaluation context; store evaluation data; verify administrator privileges; report personal-data-bearing Account IDs to Atlassian's Personal Data Reporting API on a weekly schedule. No write scope is requested. |

---

## 12. Changes to This Statement

We may update this Cloud Security Statement to reflect improvements in our security practices or changes in the Product. Updates will be reflected in the "Last Updated" date and version number.

| Version | Date | Summary |
|---------|------|---------|
| 1.4 | October 25, 2026 | Aligned with the shipped App: removed `write:jira-work`; replaced the server-side rate-limiting and automatic retention claims with the actual platform/client controls and retention triggers; corrected the admin-verification mechanism, audit event list and data-subject-rights description; added the independent-assessment status. |
| 1.3 | May 28, 2026 | Rebrand to Quality Scorecard for Jira. |
| 1.2 | May 21, 2026 | Added Personal Data Reporting (§4.5). |
| 1.0 | March 7, 2026 | Initial version. |

---

## 13. Contact

For security questions, vulnerability reports, or compliance inquiries:

| | |
|---|---|
| **Provider** | Nexus Frontier |
| **Security Contact** | Nexus Frontier Support |
| **Email** | support@nexusfrontier.net |
| **Subject Line** | Quality Scorecard for Jira — Security Inquiry |

---

*Document Version: 1.4 | Effective: March 7, 2026 | Last Updated: October 25, 2026*
