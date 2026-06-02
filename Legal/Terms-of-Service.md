# Quality Scorecard for Jira — Provider-Specific Terms

**Effective Date:** March 7, 2026
**Last Updated:** March 7, 2026
**Version:** 1.0

---

## Overview

These Provider-Specific Terms supplement the [Bonterms Standard End User Agreement (Version 1.0)](https://www.atlassian.com/licensing/marketplace/end-user-agreement-v1) ("Standard Agreement") for the Quality Scorecard for Jira application ("Product") provided by Nexus Frontier ("Provider") through the Atlassian Marketplace.

Together, these Provider-Specific Terms and the Standard Agreement form the complete Agreement between Provider and Customer. In the event of a conflict, these Provider-Specific Terms take precedence over the Standard Agreement, as set forth in Section 1.4 of the Standard Agreement.

Capitalized terms not defined herein have the meanings given to them in the Standard Agreement.

---

## 1. Product Description

**Quality Scorecard for Jira** is a Jira Cloud application built on the Atlassian Forge platform that provides quality assurance evaluation, scoring, and analytics capabilities for Jira tickets. The Product enables organizations to:

- Evaluate Jira tickets against configurable quality criteria
- Track evaluator and agent performance through analytics dashboards
- Manage evaluation workflows with role-based access control
- Generate quality coverage reports and trend analysis
- Export evaluation data for external reporting

---

## 2. Governing Law and Courts

Pursuant to Section 19.2 of the Standard Agreement:

- **Governing Law:** The laws of the Republic of Portugal.
- **Courts:** The courts of Lisbon, Portugal shall have exclusive jurisdiction.

---

## 3. Provider Notice Address

Pursuant to Section 19.3 of the Standard Agreement, Provider's notice address is:

| | |
|---|---|
| **Provider** | Nexus Frontier |
| **Attention** | Legal Department |
| **Email** | support@nexusfrontier.net |
| **Subject Line** | Quality Scorecard for Jira — Legal Notice |

---

## 4. Data Processing Agreement

Pursuant to Section 3.3 of the Standard Agreement, the parties shall adhere to the Data Processing Agreement (DPA) available at:

- [Quality Scorecard for Jira Data Processing Agreement](./Data-Processing-Agreement.md)

The DPA governs Provider's processing of Customer Data that constitutes personal data under applicable data protection laws, including the GDPR.

---

## 5. Security Measures

Pursuant to Section 3.2 of the Standard Agreement, Provider implements the following Security Measures:

### 5.1 Infrastructure Security

- All Customer Data is stored within Atlassian Forge Key-Value Storage, hosted and managed by Atlassian.
- Provider does not operate any external servers or infrastructure for data storage.
- Encryption at rest (AES-256) and in transit (TLS 1.2+) is provided by Atlassian's infrastructure.

### 5.2 Application Security

- Authentication is handled natively by the Atlassian Forge platform.
- Role-based access control (RBAC) with per-project granularity.
- Input validation and sanitization on all user-supplied data.
- Per-user, per-endpoint rate limiting.
- No external data egress — the Product makes no outbound requests outside Atlassian's infrastructure.

### 5.3 Operational Security

- Regular dependency audits and security scans.
- Automated vulnerability scanning through Atlassian's Ecoscanner.
- No hardcoded secrets, API keys, or credentials in source code.
- Audit logging of administrative actions.

For complete details, see the [Cloud Security Statement](./Cloud-Security-Statement.md).

---

## 6. Support Policy

Pursuant to Section 5.1 of the Standard Agreement, Provider offers the following support:

### 6.1 Support Channels

| Channel | Details |
|---------|---------|
| **Email** | support@nexusfrontier.net |
| **Marketplace Listing** | Via the Quality Scorecard for Jira Marketplace page |

### 6.2 Support Scope

Support covers:
- Installation and configuration assistance
- Bug reports and defect resolution
- General usage questions
- Data export and GDPR requests

Support does not cover:
- Customization or bespoke feature development
- Issues caused by third-party apps or Customer modifications
- Jira Cloud platform issues (directed to Atlassian Support)

### 6.3 Response Times

| Severity | Description | Target Response |
|----------|-------------|-----------------|
| Critical | Product entirely unavailable | 24 hours |
| High | Major feature non-functional | 48 hours |
| Medium | Non-critical feature issue or degraded performance | 5 business days |
| Low | General questions, minor issues, feature requests | 10 business days |

Response times are targets and not guaranteed service levels. Support is provided during business hours (Monday–Friday, 9:00–18:00 WET/WEST, excluding Portuguese public holidays).

---

## 7. Acceptable Use Policy

Pursuant to Section 7.1 of the Standard Agreement, Customer agrees to the following Acceptable Use Policy:

### 7.1 Permitted Use

The Product is intended for organizational quality assurance evaluation of Jira tickets. Customer may use the Product for:
- Evaluating ticket quality against defined criteria
- Generating quality analytics and reports
- Managing evaluator workflows and assignments
- Exporting evaluation data for internal reporting

### 7.2 Prohibited Use

Customer shall not:
- Use the Product to store Sensitive Data (as defined in the Standard Agreement) in evaluation feedback fields
- Attempt to circumvent rate limits or access controls
- Use the Product's evaluation features for purposes unrelated to quality assurance (e.g., employee disciplinary records)
- Reverse engineer, decompile, or attempt to extract the source code of the Product
- Use the Product in a manner that violates Atlassian's Acceptable Use Policy

---

## 8. Usage Data

Pursuant to Section 3.4 of the Standard Agreement, Provider may collect the following Usage Data:

- Aggregate feature usage statistics (e.g., number of evaluations created, dashboard views)
- Error and performance telemetry generated by the Atlassian Forge platform
- App installation and configuration metadata

Usage Data does not include Customer Data or personal data. Provider uses Usage Data solely to improve the Product and does not share it with third parties.

---

## 9. Product-Specific Terms

### 9.1 Data Storage

All Customer Data is stored exclusively within Atlassian Forge Key-Value Storage. Provider does not store Customer Data on any infrastructure outside of Atlassian's platform.

### 9.2 Evaluation Data Ownership

Evaluation data (scores, feedback, and associated metadata) created by Customer's Users is Customer Data as defined in the Standard Agreement. Customer retains all rights to this data.

### 9.3 Data Export

Customer may export evaluation data at any time through the Product's built-in export functionality, which supports Excel (XLSX) and PDF formats.

### 9.4 Data Deletion

Upon Customer's request, Provider will delete specific evaluation data or all Customer Data associated with a particular user. Customer administrators can also perform data deletion through the Product's GDPR compliance features.

### 9.5 Availability

Product availability is dependent on the availability of the Atlassian Forge platform and Jira Cloud. Provider does not offer an independent SLA, as the Product's uptime is determined by Atlassian's infrastructure availability.

---

## 10. Limitation of Liability — Additional Terms

### 10.1 General Cap

For the purposes of Section 14.1 of the Standard Agreement, the "General Cap" is the total amounts paid by Customer to Provider for the Product in the twelve (12) months immediately preceding the first incident giving rise to liability, as defined in the Standard Agreement.

### 10.2 Platform Disclaimer

Provider is not liable for:
- Downtime, data loss, or performance issues caused by the Atlassian Forge platform or Jira Cloud
- Changes to Atlassian APIs or platform behavior that affect Product functionality
- Customer's failure to maintain adequate Jira Cloud permissions for the Product to function

---

## 11. Modifications to These Terms

Provider may update these Provider-Specific Terms to reflect changes in the Product, applicable laws, or business practices. Material changes will be communicated through the Marketplace listing. Continued use of the Product after changes constitutes acceptance of the updated terms.

---

## 12. Entire Agreement

These Provider-Specific Terms, together with the [Bonterms Standard End User Agreement (Version 1.0)](https://www.atlassian.com/licensing/marketplace/end-user-agreement-v1) and the [Data Processing Agreement](./Data-Processing-Agreement.md), constitute the entire Agreement between Provider and Customer regarding the Product.

---

## Contact

| | |
|---|---|
| **Provider** | Nexus Frontier |
| **Email** | support@nexusfrontier.net |

---

*These Provider-Specific Terms are provided as a legal framework. Nexus Frontier recommends that Customers consult their own legal counsel before agreeing to these terms.*

*Document Version: 1.0 | Effective: March 7, 2026*
