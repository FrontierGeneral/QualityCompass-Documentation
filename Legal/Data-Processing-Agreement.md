# Quality Scorecard for Jira — Data Processing Agreement (DPA)

**Effective Date:** March 7, 2026
**Last Updated:** October 25, 2026
**Version:** 1.3

---

## 1. Introduction and Scope

This Data Processing Agreement ("DPA") is entered into between the organization that installs Quality Scorecard for Jira ("Customer," "Controller") and Nexus Frontier ("Provider," "Processor"), and supplements the Agreement between Customer and Provider for the Quality Scorecard for Jira application ("Product").

This DPA applies to the processing of Personal Data by Provider on behalf of Customer in connection with Customer's use of the Product. This DPA is governed by and forms part of the Agreement, including the Provider-Specific Terms.

---

## 2. Definitions

In this DPA, the following terms have the meanings set out below. Capitalized terms not defined herein have the meanings given in the Agreement, the Standard Agreement, or applicable Data Protection Laws.

- **"Data Protection Laws"** means all applicable laws and regulations relating to the processing of Personal Data, including (a) the General Data Protection Regulation (EU 2016/679) ("GDPR"), (b) the UK General Data Protection Regulation ("UK GDPR"), (c) the California Consumer Privacy Act ("CCPA"), and (d) any applicable national implementing legislation.
- **"Personal Data"** means any information relating to an identified or identifiable natural person that is processed by Provider on behalf of Customer in connection with the Product.
- **"Processing"** means any operation performed on Personal Data, including collection, recording, organization, structuring, storage, adaptation, alteration, retrieval, consultation, use, disclosure, combination, restriction, erasure, or destruction.
- **"Data Subject"** means an identified or identifiable natural person to whom Personal Data relates.
- **"Sub-processor"** means a third party engaged by Provider to process Personal Data on behalf of Customer.
- **"Security Incident"** means any accidental or unlawful destruction, loss, alteration, unauthorized disclosure of, or access to, Personal Data.

---

## 3. Roles and Responsibilities

### 3.1 Customer as Controller

Customer determines the purposes and means of processing Personal Data through its use of the Product. Customer is responsible for:
- Ensuring a lawful basis for processing Personal Data through the Product
- Providing appropriate notices to Data Subjects
- Responding to Data Subject requests (with Provider's assistance as described herein)
- Ensuring that its use of the Product complies with applicable Data Protection Laws

### 3.2 Provider as Processor

Provider processes Personal Data solely on behalf of and under the documented instructions of Customer. Provider is responsible for:
- Processing Personal Data only as instructed by Customer (as documented in this DPA and the Agreement)
- Implementing appropriate technical and organizational measures to protect Personal Data
- Assisting Customer in fulfilling its obligations under Data Protection Laws
- Notifying Customer of any Security Incidents

---

## 4. Details of Processing

### 4.1 Subject Matter and Duration

| Element | Description |
|---------|-------------|
| **Subject Matter** | Processing of Personal Data in connection with quality assurance evaluations of Jira tickets |
| **Duration** | For the duration of the Agreement, plus any post-termination retention period described in Section 10 |
| **Nature of Processing** | Storage, retrieval, display, aggregation, analysis, and deletion of Personal Data within Atlassian Forge infrastructure |

### 4.2 Categories of Data Subjects

- Employees and contractors of Customer who use the Product as evaluators
- Employees and contractors of Customer whose Jira tickets are evaluated (agents/assignees)
- Employees and contractors of Customer who administer the Product

### 4.3 Types of Personal Data Processed

| Personal Data | Source | Processing Purpose |
|---------------|--------|-------------------|
| Jira Account ID | Jira Cloud API | Unique identification of users within evaluations, roles, and audit logs |
| Display Name | Jira Cloud API | Human-readable identification of evaluators and agents in the UI and reports |
| Email Address | Jira Cloud API | Read only as a last-resort fallback for the current user's own display name; never stored or displayed to others |
| Avatar URL | Jira Cloud API | Display user profile images in the Product UI (not stored) |
| Jira Permission Results | Jira Cloud API | Determine administrator privileges (global Administer Jira permission) and project browse rights |
| Group Memberships | Jira Cloud API | Only when Customer configures an optional administrator group |
| Evaluation Authorship and Feedback | Product-generated | Record which user created each evaluation (Account ID + Display Name) together with the free-text feedback they entered |
| Role Assignments | Product-generated | Record which users are assigned to which roles per project |
| Administrative Actions | Product-generated | Audit trail of denied access attempts, role/permission changes, evaluation deletions, project access changes and personal-data erasures |
| Display Name Caches | Product-generated | Account ID → Display Name caches (manual overrides and automatically resolved names) |
| Personal Data Index | Product-generated | Atlassian Account IDs and last-updated timestamps for users whose data the Product stores. Reported weekly to Atlassian's Personal Data Reporting API so Atlassian can notify the Product when an account is closed or its profile is updated. |

### 4.4 Sensitive Data

Provider does not intentionally process any special categories of Personal Data (as defined in Article 9 of the GDPR) or Sensitive Data (as defined in the Standard Agreement). Customer shall not submit Sensitive Data to the Product.

---

## 5. Provider Obligations

### 5.1 Processing Instructions

Provider shall:
- Process Personal Data only on Customer's documented instructions (including those in this DPA and the Agreement), unless required by applicable law
- Promptly inform Customer if, in Provider's opinion, an instruction infringes Data Protection Laws

### 5.2 Confidentiality

Provider shall ensure that persons authorized to process Personal Data:
- Have committed to confidentiality obligations or are under an appropriate statutory obligation of confidentiality
- Process Personal Data only as instructed

### 5.3 Security Measures

Provider implements the following technical and organizational measures to protect Personal Data:

**Technical Measures:**

| Measure | Implementation |
|---------|----------------|
| Encryption at Rest | AES-256 via Atlassian Forge infrastructure |
| Encryption in Transit | TLS 1.2+ for all communications |
| Access Control | Atlassian Forge native authentication; no separate credentials |
| Authorization | Role-based access control (RBAC) with per-project granularity, enforced server-side; client storage access restricted to an allow-list of keys |
| Input Validation | Backend validation of all resolver inputs (JQL, project keys, issue ids, account ids, storage keys, pagination) |
| Abuse Protection | Atlassian platform rate limits plus client-side concurrency limiting and request de-duplication; denied access attempts are audited |
| Network Isolation | No external data egress; all processing within Atlassian infrastructure |

**Organizational Measures:**

| Measure | Implementation |
|---------|----------------|
| Least Privilege | App requests minimum required Jira API scopes |
| Audit Logging | Administrative actions and data changes logged with user identity and timestamp |
| Dependency Management | Regular security audits of software dependencies |
| Code Review | Security-focused code reviews before releases |
| Vulnerability Scanning | Automated scanning via Atlassian Ecoscanner |

### 5.4 Sub-processors

#### 5.4.1 Authorized Sub-processors

Customer provides general authorization for Provider to engage the following Sub-processor:

| Sub-processor | Purpose | Location |
|---------------|---------|----------|
| Atlassian Pty Ltd | Cloud infrastructure (Forge platform), data storage (Forge Key-Value Storage), and Jira Cloud APIs | Per Atlassian's data residency settings (see [Atlassian Data Residency](https://support.atlassian.com/security-and-access-policies/docs/understand-data-residency/)) |

#### 5.4.2 Changes to Sub-processors

Provider shall notify Customer before engaging any new Sub-processor by updating this DPA and providing notice through the Marketplace listing. Customer may object to a new Sub-processor by providing written notice within 30 days. If Provider cannot reasonably accommodate the objection, Customer may terminate the Agreement.

### 5.5 Assistance with Data Subject Rights

Provider shall assist Customer in responding to Data Subject requests by:
- Providing export functionality in the Product's dashboards (CSV/Excel) through which Customer administrators can extract evaluation data for a Data Subject (Right of Access, Right to Data Portability), and by compiling any additional Personal Data (audit entries, caches, assignments) on Customer's written request
- Providing deletion functionality for Customer administrators (deletion of a ticket's evaluations; removal of role assignments) and performing complete erasure of a named Data Subject's Personal Data from all storage locations on Customer's written request, within 30 days (Right to Erasure)
- Supporting data rectification through Jira profile updates and superseding evaluations (Right to Rectification)
- Enabling access restriction through RBAC configuration (Right to Restriction)
- Participating in Atlassian's Personal Data Reporting API: on a weekly schedule, the Product reports its Personal Data Index to Atlassian. When Atlassian indicates that a Data Subject's Atlassian account has been closed, the Product automatically erases that Data Subject's evaluations from the site-wide evaluation list and removes the related name override and index entry; residual references in project-scoped lists, role assignments, automatic name caches and audit entries are removed by Provider on request or upon uninstallation. When Atlassian indicates that a Data Subject's profile has been updated, the Product refreshes its cached display name from Atlassian's user APIs.

The Product does not provide a self-service export or erasure function to individual end users; requests are routed through Customer's administrators.

Provider shall promptly inform Customer if it receives a request directly from a Data Subject, without responding to the request unless authorized by Customer.

### 5.6 Assistance with Compliance

Provider shall assist Customer, taking into account the nature of processing and information available to Provider, with:
- Data protection impact assessments (Article 35 GDPR)
- Prior consultation with supervisory authorities (Article 36 GDPR)
- Compliance with security obligations (Article 32 GDPR)

---

## 6. Security Incident Notification

### 6.1 Notification

Provider shall notify Customer without undue delay (and in any event within 72 hours) after becoming aware of a Security Incident affecting Personal Data processed under this DPA.

### 6.2 Notification Content

The notification shall include, to the extent available:
- A description of the nature of the Security Incident, including the categories and approximate number of Data Subjects and Personal Data records affected
- The name and contact details of Provider's contact point
- A description of the likely consequences of the Security Incident
- A description of the measures taken or proposed to address the Security Incident, including mitigation measures

### 6.3 Cooperation

Provider shall cooperate with Customer and take reasonable steps to assist in the investigation, mitigation, and remediation of each Security Incident.

---

## 7. International Data Transfers

### 7.1 Transfer Mechanism

Provider does not independently transfer Personal Data outside the European Economic Area (EEA). All Personal Data is stored within Atlassian Forge infrastructure, and any international transfers are governed by Atlassian's data transfer mechanisms, which include:
- Standard Contractual Clauses (SCCs) approved by the European Commission
- Additional safeguards as required by applicable law

### 7.2 Provider's Location

Provider's personnel who may access Personal Data for support purposes are located in Portugal (within the EEA).

---

## 8. Audits

### 8.1 Information and Audit Rights

Provider shall make available to Customer all information necessary to demonstrate compliance with this DPA and allow for and contribute to audits, including inspections, conducted by Customer or Customer's authorized auditor.

### 8.2 Scope

Given that all Personal Data is stored within Atlassian's Forge infrastructure (not on Provider's servers), audits shall focus on:
- Provider's application code and security practices
- Provider's organizational measures and policies
- Provider's Sub-processor agreements and compliance
- Records of Processing activities

### 8.3 Conditions

- Customer shall provide reasonable advance notice (at least 30 days) for audits.
- Audits shall be conducted during business hours and shall not unreasonably interfere with Provider's operations.
- Customer shall bear the costs of any audit.
- Audit frequency shall be limited to once per year, unless required by a supervisory authority or following a Security Incident.

---

## 9. Records of Processing Activities

Provider maintains a record of processing activities carried out on behalf of Customer, including:
- The name and contact details of Provider (as Processor) and Customer (as Controller)
- The categories of processing carried out on behalf of Customer
- Transfers of Personal Data to third countries (if any) and the transfer mechanism
- A general description of the technical and organizational security measures

---

## 10. Data Retention and Deletion

### 10.1 During the Agreement

The Product does not apply automatic time-based expiry to Personal Data. Personal Data is retained for the duration of the Agreement and is removed through the mechanisms below (see also the Privacy Policy, Section 7):

| Data Category | Retained until |
|---------------|-----------------|
| Ticket Evaluations | Deleted by an authorised Customer user, erased following Atlassian account closure, erased by Provider on Customer's request, or app uninstalled |
| Role Definitions and Assignments | Changed by a Customer administrator or app uninstalled |
| Audit Logs | App uninstalled (a 90-day automatic retention is planned but not yet enforced) |
| Display Name Caches and Personal Data Index | Account closure, Provider erasure on request, or app uninstalled; server-side profile caches refresh after 1–24 hours |
| Configuration and User Preferences | Changed by the user/administrator or app uninstalled |
| Client-side cache data | 1 minute – 2 hours (browser) |

### 10.2 Upon Termination

Upon termination or expiration of the Agreement, Provider shall:
- Within 60 days of Customer's request, delete or return all Personal Data processed under this DPA
- Delete existing copies of Personal Data unless required by applicable law to retain them

Deletion of Personal Data stored in Forge Key-Value Storage is handled through the Atlassian platform upon app uninstallation. Atlassian retains uninstalled-app data for up to 28 days as a soft-delete grace period (during which the data is inaccessible to the Product but recoverable if Customer reinstalls within 21 days); after 28 days the data is permanently deleted by the Atlassian platform. For data deletion prior to uninstallation, Customer may use the Product's administrator deletion functions or contact Provider.

---

## 11. Liability

Each party's liability arising from or related to this DPA is subject to the limitations of liability set forth in the Agreement, including the Enhanced Cap for breaches of Security Measures (Section 3.2 of the Standard Agreement) as specified in Section 14.3 of the Standard Agreement.

---

## 12. Governing Law

This DPA is governed by the same governing law as the Agreement (the laws of the Republic of Portugal), unless otherwise required by Data Protection Laws.

---

## 13. Contact

For any matters related to this DPA:

| | |
|---|---|
| **Provider** | Nexus Frontier |
| **Email** | support@nexusfrontier.net |
| **Subject Line** | Quality Scorecard for Jira — DPA Inquiry |

---

## Annex A: Technical and Organizational Security Measures

This Annex describes the technical and organizational security measures implemented by Provider pursuant to Article 32 of the GDPR.

### A.1 Measures of Pseudonymization and Encryption

- All data at rest is encrypted using AES-256 (provided by Atlassian Forge infrastructure)
- All data in transit is encrypted using TLS 1.2 or higher
- User identification within evaluations uses Jira Account IDs (pseudonymous identifiers)

### A.2 Measures for Ensuring Ongoing Confidentiality, Integrity, Availability, and Resilience

- Multi-tenant isolation provided by Atlassian Forge (data is isolated per app per site)
- Input validation and sanitization prevent data corruption
- Atlassian platform rate limits and client-side concurrency limiting mitigate denial-of-service conditions
- Atlassian infrastructure provides high availability and disaster recovery

### A.3 Measures for Ensuring the Ability to Restore Availability and Access to Personal Data

- Data stored in Atlassian Forge infrastructure benefits from Atlassian's backup and recovery mechanisms
- Built-in data export functionality allows Customer to maintain offline copies

### A.4 Processes for Regularly Testing, Assessing, and Evaluating Effectiveness

- Regular dependency audits using `npm audit`
- Automated vulnerability scanning via Atlassian Ecoscanner
- Security-focused code reviews
- Forge linting (`forge lint`) for manifest and code compliance

### A.5 Measures for User Identification and Authorization

- Authentication handled natively by Atlassian Forge — users are authenticated via Atlassian SSO
- Role-based access control (RBAC) with per-project granularity, enforced in every backend function
- Administrator verification via Jira's global Administer Jira permission (optionally supplemented by a Customer-designated Jira group)
- No separate user accounts or credentials required

### A.6 Measures for the Protection of Data During Transmission

- All API communications between the Product and Jira Cloud occur via Forge's internal runtime (no external network transit)
- No data is transmitted outside Atlassian's infrastructure
- Client-to-Forge communications use HTTPS with Atlassian-managed TLS

### A.7 Measures for the Protection of Data During Storage

- Forge Key-Value Storage provides encryption at rest
- Data is logically isolated per Jira site and per app installation
- Client-side caches (IndexedDB) are stored locally in the user's browser and expire automatically

### A.8 Measures for Ensuring Physical Security of Locations at Which Personal Data Are Processed

- All server-side processing occurs within Atlassian's data centers, which maintain SOC 2, ISO 27001, and ISO 27018 certifications
- Provider does not maintain physical infrastructure for data processing

### A.9 Measures for Ensuring Events Logging

- Denied access attempts, role/permission changes, evaluation deletions and personal-data erasures are logged with user identity, timestamp, and event type; project access changes are logged separately and shown in the Product
- Audit entries are retained for the life of the installation (a 90-day automatic retention is planned)
- Security-relevant events (access denied, data deletion) are flagged with elevated severity

### A.10 Measures for Ensuring System Configuration

- App manifest (`manifest.yml`) enforces least-privilege API scopes (currently `read:jira-work`, `storage:app`, `read:jira-user`, `report:personal-data`; no write scope)
- The Product makes no external egress to third-party services. The only outbound network call leaves Atlassian's runtime is to Atlassian's own Personal Data Reporting API at `api.atlassian.com`, performed via the `privacy.reportPersonalData` helper in `@forge/api` (which handles app authentication and batching internally)
- Forge runtime enforces execution time limits and payload size restrictions

---

*This DPA is provided as a legal framework for data processing compliance. Nexus Frontier recommends that both parties consult their own legal counsel regarding their obligations under applicable Data Protection Laws.*

**Change history:** 1.3 (October 25, 2026) — aligned data inventory, security measures, data-subject assistance and retention with the shipped Product; removed `write:jira-work`, consent records and per-user rate limiting. 1.2 (May 28, 2026) — rebrand. 1.1 (May 27, 2026) — Personal Data Reporting. 1.0 (March 7, 2026) — initial.

*Document Version: 1.3 | Effective: March 7, 2026 | Last Updated: October 25, 2026*
