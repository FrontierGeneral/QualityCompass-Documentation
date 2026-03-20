---
layout: page
title: Cloud Security
permalink: /cloud-security/
---

# Quality Compass — Cloud Security Statement

**Effective Date:** March 7, 2026
**Last Updated:** March 7, 2026
**Version:** 1.0

---

## Architecture

Quality Compass is built entirely on **Atlassian Forge** — Atlassian's serverless, cloud-native platform. This means:

- **No self-hosted infrastructure** — runs on Atlassian's managed infrastructure
- **No external services** — zero outbound network connections
- **Sandboxed execution** — isolated Forge Function environment
- **Managed runtime** — Node.js 20.x, patched by Atlassian

---

## Data Storage

All data stored in **Atlassian Forge Key-Value Storage**:

| Property | Implementation |
|----------|---------------|
| Encryption at Rest | AES-256 |
| Encryption in Transit | TLS 1.2+ |
| Tenant Isolation | Per app, per Jira Cloud site |
| Physical Security | SOC 2, ISO 27001, ISO 27018 certified data centers |

---

## Authentication & Authorization

| Layer | Mechanism |
|-------|-----------|
| **Platform** | Atlassian SSO — no separate credentials |
| **Admin** | Jira group membership verification |
| **RBAC** | Per-project role-based access control |
| **Data Scoping** | Project-level isolation |

API scopes (least privilege): `read:jira-work`, `write:jira-work`, `storage:app`, `read:jira-user`

---

## Application Security

| Vulnerability | Mitigation |
|---------------|-----------|
| XSS | Output encoding, CSP, sandboxed iframe |
| CSRF | Forge platform native protection |
| Injection | Input validation, parameterized API calls |
| CSV Injection | Character prefixing in exports |
| Prototype Pollution | `Object.create(null)` for untrusted data |
| ReDoS | Optimized regex patterns |

---

## Data Protection

- GDPR-ready: built-in data export, deletion, and consent management
- Data retention: 90 days (audit logs) to 365 days (evaluations)
- No data shared with third parties
- No external analytics or tracking

---

## Incident Response

1. Detection via audit logs, Forge monitoring, or user reports
2. Severity classification and impact analysis
3. Containment and mitigation
4. Customer notification within 72 hours (per GDPR Article 33)
5. Remediation and post-incident review

---

## Compliance

Atlassian infrastructure certifications: SOC 2 Type II, ISO 27001, ISO 27018, CSA STAR.

---

## Marketplace Privacy & Security Tab

| Question | Answer |
|----------|--------|
| Data stored outside Atlassian? | **No** |
| Data processed outside Atlassian? | **No** |
| Data shared with third parties? | **No** |
| GDPR role | Processor (Customer is Controller) |
| DPA available? | **Yes** — [Data Processing Agreement](./dpa) |
| Encryption at rest? | **Yes** — AES-256 |

---

## Contact

**Email:** [support@nexusfrontier.net](mailto:support@nexusfrontier.net)
**Subject:** Quality Compass — Security Inquiry

*The full Cloud Security Statement is available in our [Legal documentation](./Legal/Cloud-Security-Statement.md).*

*Document Version: 1.0 | Effective: March 7, 2026*
