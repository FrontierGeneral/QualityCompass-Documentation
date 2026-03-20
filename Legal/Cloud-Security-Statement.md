# Quality Compass — Cloud Security Statement

**Effective Date:** March 7, 2026
**Last Updated:** March 7, 2026
**Version:** 1.0

---

## 1. Overview

This Cloud Security Statement describes the security practices, architecture, and controls implemented for the Quality Compass application ("the App"), a Jira Cloud application built on the Atlassian Forge platform.

Quality Compass is designed with a security-first approach, leveraging Atlassian's enterprise-grade infrastructure for all data storage, authentication, and runtime execution.

---

## 2. Architecture and Hosting

### 2.1 Platform

Quality Compass is built entirely on the **Atlassian Forge** platform, Atlassian's serverless, cloud-native app development framework. This means:

- **No self-hosted infrastructure:** The App runs on Atlassian's managed infrastructure (AWS Lambda). Nexus Frontier does not operate, manage, or have access to any servers.
- **No external services:** The App does not communicate with any servers, APIs, or services outside of Atlassian's infrastructure.
- **Sandboxed execution:** Forge Functions execute in an isolated, sandboxed environment managed by Atlassian.
- **Managed runtime:** Node.js 20.x, managed and patched by Atlassian.

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
│  │  Jira Cloud   │◄──►│  Quality Compass (Forge App)   │  │
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
│                  NO EXTERNAL CONNECTIONS                   │
└──────────────────────────────────────────────────────────┘
```

**Key point:** There are zero outbound network connections from the App to any external domain. The `manifest.yml` declares empty external fetch permissions:

```yaml
permissions:
  external:
    fetch:
      backend: []
      client: []
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
| **Admin Verification** | Jira Group Membership | Administrative functions verify the user belongs to the configured admin group via the Jira permissions API |
| **RBAC** | App-Level Roles | Per-project role-based access control with configurable permissions (view analytics, create evaluations, manage settings, etc.) |
| **Data Scoping** | Project-Level Isolation | Users can only access evaluations for projects they have been granted access to |

### 3.3 API Scopes (Least Privilege)

The App requests only the minimum Jira API scopes necessary:

| Scope | Justification |
|-------|---------------|
| `read:jira-work` | Read issue data (key, summary, status, priority, assignee) for evaluation context |
| `write:jira-work` | Reserved for future Jira issue updates; not currently used for direct writes |
| `storage:app` | Store and retrieve evaluations, criteria, roles, and configuration in Forge Storage |
| `read:jira-user` | Read user profiles (Account ID, Display Name, group membership) for identification and admin verification |

---

## 4. Data Protection

### 4.1 Data Classification

| Classification | Data Types | Protection Level |
|---------------|------------|-----------------|
| **Confidential** | Jira Account IDs, Email Addresses | High — encrypted at rest and in transit, access controlled |
| **Internal** | Evaluation scores, feedback, audit logs | Standard — encrypted at rest and in transit |
| **Public** | Evaluation criteria definitions, coverage targets | Standard — no personal data |

### 4.2 Personal Data Handling

| Principle | Implementation |
|-----------|---------------|
| **Data Minimization** | Only data necessary for evaluation functionality is collected |
| **Purpose Limitation** | Data is used solely for quality assurance evaluation purposes |
| **Storage Limitation** | Defined retention periods enforced per data category |
| **Integrity and Confidentiality** | Encryption at rest and in transit; input validation; access controls |

### 4.3 Data Retention

| Category | Period | Auto-Enforcement |
|----------|--------|-----------------|
| Evaluations | 365 days | Via GDPR service |
| Audit Logs | 90 days | Via GDPR service |
| User Preferences | 730 days | Via GDPR service |
| Analytics Data | 180 days | Via GDPR service |
| Temporary/Cache | 7 days (server) / auto-expiring (client) | Automatic |

### 4.4 Data Subject Rights

The App provides built-in tools for GDPR compliance:

- **Data Export:** Compile all personal data for a specific user into a structured export
- **Data Deletion:** Remove all personal data for a specific user across evaluations, audit logs, preferences, and role assignments
- **Consent Management:** Record and manage user consent decisions with timestamps

---

## 5. Application Security

### 5.1 Input Validation

All user-supplied input is validated and sanitized before processing:

| Input Type | Validation |
|------------|-----------|
| JQL Queries | Pattern-based injection prevention (SQL comments, statement separators, union attacks, XSS, JavaScript protocol) |
| Project Keys | Alphanumeric pattern enforcement, maximum length |
| Account IDs | Format validation with length limits |
| Feedback Text | HTML entity encoding, length limits |
| Evaluation Scores | Integer range validation (0–10) |
| Storage Keys | Alphanumeric pattern with allowed separators, length limits |

### 5.2 Common Vulnerability Protections

| Vulnerability | Mitigation |
|---------------|-----------|
| **Cross-Site Scripting (XSS)** | Output encoding, Content Security Policy (CSP) enforced by Forge, sandboxed iframe |
| **Cross-Site Request Forgery (CSRF)** | Forge platform handles CSRF protection natively |
| **Injection Attacks** | Input validation and parameterized Jira API calls |
| **CSV Injection** | Dangerous character prefixing in exported values |
| **Prototype Pollution** | Use of `Object.create(null)` for untrusted data structures |
| **Regular Expression DoS** | Optimized regex patterns |
| **Sensitive Data in Logs** | No `console.log` statements exposing personal data; error logging uses sanitized messages |

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

### 6.1 Server-Side Rate Limiting

- Per-user, per-endpoint rate limiting tracked in Forge Storage
- Request counts reset on configurable time windows
- Rate limit exceeded events are logged to the audit service

### 6.2 Client-Side Controls

- Concurrency limiter (default: 8 in-flight requests) reduces contention
- In-flight request deduplication coalesces identical API calls
- Exponential backoff with jitter on HTTP 429 responses

---

## 7. Audit and Monitoring

### 7.1 Audit Logging

Administrative and security-relevant events are logged:

| Event Category | Examples | Severity |
|---------------|----------|----------|
| **Access Control** | Access granted, access denied | INFO / CRITICAL |
| **Data Operations** | Evaluation created, data exported, data deleted | INFO / CRITICAL |
| **Administrative** | Settings changed, project access changed, permission changed | INFO |
| **Security** | Rate limit exceeded, invalid input, suspicious activity | WARNING / CRITICAL |
| **GDPR** | Consent updated, user data deleted, retention enforced | INFO / CRITICAL |

### 7.2 Audit Log Retention

Audit logs are retained for 90 days. Security alerts are stored with elevated priority.

### 7.3 Forge Platform Logs

The Atlassian Forge platform independently provides:
- Function invocation logs (accessible via `forge logs`)
- Error and exception tracking
- Performance metrics

---

## 8. Vulnerability Management

### 8.1 Dependency Management

- Dependencies are regularly audited using `npm audit`
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
| **Subject Line** | Quality Compass — Security Incident |

---

## 10. Compliance

### 10.1 Atlassian Infrastructure Compliance

Quality Compass benefits from Atlassian's infrastructure certifications:

| Certification | Scope |
|--------------|-------|
| SOC 2 Type II | Security, Availability, Confidentiality |
| ISO 27001 | Information Security Management |
| ISO 27018 | Protection of PII in Public Clouds |
| CSA STAR | Cloud Security |

### 10.2 App-Level Compliance

| Requirement | Status |
|-------------|--------|
| GDPR Data Subject Rights | Implemented (export, delete, consent management) |
| Data Minimization | Only necessary data collected |
| Least Privilege Scopes | Minimum Jira API scopes requested |
| Encryption at Rest | AES-256 via Forge infrastructure |
| Encryption in Transit | TLS 1.2+ |
| No External Data Sharing | Zero third-party data transfers |
| Audit Logging | Administrative and security events logged |
| Input Validation | All user inputs validated and sanitized |
| Secure Development Practices | Code review, dependency auditing, automated scanning |

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
| Does the app store data after uninstallation? | Data in Forge Storage is handled per Atlassian's uninstallation lifecycle. The App does not maintain any independent data store. |
| Does the app allow custom data retention periods? | The App enforces predefined retention periods (90 days to 730 days depending on data category). |
| Is the provider a "data controller" under GDPR? | **No.** Nexus Frontier acts as a Data Processor. The Customer is the Data Controller. |
| Is the provider a "data processor" under GDPR? | **Yes.** With respect to all End-User Data processed through the App. |
| Does the app have a DPA? | **Yes.** Available at [Data Processing Agreement](./Data-Processing-Agreement.md). |
| Does the app transfer EEA data outside the EEA? | **No.** The App does not independently transfer data. Atlassian manages any cross-border transfers. |
| Does the app use full disk encryption at rest? | **Yes.** Provided by Atlassian Forge infrastructure (AES-256). |
| Does the app access PATs or shared secrets? | **No.** Authentication is handled natively by Forge. |
| API scopes requested | `read:jira-work`, `write:jira-work`, `storage:app`, `read:jira-user` |
| Scope justification | Read issue and user data for evaluation context; store evaluation data; verify administrator privileges. |

---

## 12. Changes to This Statement

We may update this Cloud Security Statement to reflect improvements in our security practices or changes in the Product. Updates will be reflected in the "Last Updated" date and version number.

---

## 13. Contact

For security questions, vulnerability reports, or compliance inquiries:

| | |
|---|---|
| **Provider** | Nexus Frontier |
| **Security Contact** | Nexus Frontier Support |
| **Email** | support@nexusfrontier.net |
| **Subject Line** | Quality Compass — Security Inquiry |

---

*Document Version: 1.0 | Effective: March 7, 2026*
