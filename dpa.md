---
layout: page
title: Data Processing Agreement
permalink: /dpa/
---

# Quality Scorecard for Jira — Data Processing Agreement (DPA)

**Effective Date:** March 7, 2026
**Last Updated:** March 7, 2026
**Version:** 1.0

---

## 1. Scope

This DPA supplements the Agreement between the Customer (Controller) and Nexus Frontier (Processor) for processing of Personal Data through Quality Scorecard for Jira.

---

## 2. Roles

- **Customer** is the Data Controller — determines purposes and means of processing
- **Nexus Frontier** is the Data Processor — processes data on Customer's instructions
- **Atlassian** is a Sub-processor — provides Forge infrastructure and storage

---

## 3. Data Processed

| Personal Data | Source | Purpose |
|---------------|--------|---------|
| Jira Account ID | Jira Cloud API | User identification |
| Display Name | Jira Cloud API | UI display |
| Email Address | Jira Cloud API | Fallback identification (not stored) |
| Evaluation Authorship | Product-generated | Evaluation records |
| Role Assignments | Product-generated | Access control |
| Administrative Actions | Product-generated | Audit trail |

**Data Subjects:** Customer's employees and contractors who evaluate, are evaluated, or administer the Product.

**No Sensitive Data:** The Product does not process special categories of data (GDPR Article 9).

---

## 4. Security Measures

See [Cloud Security Statement](./cloud-security). Key measures:
- AES-256 encryption at rest, TLS 1.2+ in transit
- Atlassian SSO authentication
- RBAC with per-project granularity
- Input validation and rate limiting
- Zero external data egress

---

## 5. Sub-processors

| Sub-processor | Purpose | Location |
|---------------|---------|----------|
| Atlassian Pty Ltd | Forge platform, data storage, Jira APIs | Per Atlassian data residency |

30 days notice before engaging new sub-processors.

---

## 6. Data Subject Rights

Built-in tools for:
- **Data Export** (Access, Portability)
- **Data Deletion** (Erasure)
- **Access Restriction** (via RBAC)
- **Rectification** (via Jira profile updates)

---

## 7. Incident Notification

Customer notified within **72 hours** of confirmed Security Incidents involving Personal Data.

---

## 8. Data Retention

| Data | Retention |
|------|-----------|
| Evaluations | 365 days |
| Audit Logs | 90 days |
| User Preferences | 730 days |
| Analytics | 180 days |

Upon termination: data deleted within 60 days of request.

---

## 9. International Transfers

No independent data transfers outside the EEA. All data within Atlassian Forge. Atlassian maintains Standard Contractual Clauses. Provider personnel located in Portugal (EEA).

---

## 10. Audits

Available with 30 days notice, once per year, at Customer's cost. Focus: application code, security practices, organizational measures.

---

## 11. Governing Law

Republic of Portugal, unless otherwise required by Data Protection Laws.

---

## 12. Contact

**Email:** [support@nexusfrontier.net](mailto:support@nexusfrontier.net)
**Subject:** Quality Scorecard for Jira — DPA Inquiry

*The full Data Processing Agreement is available in our [Legal documentation](./Legal/Data-Processing-Agreement.md).*

*Document Version: 1.0 | Effective: March 7, 2026*
