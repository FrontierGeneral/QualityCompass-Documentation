# Security Documentation

## Quick Navigation
- [Architecture Overview](./Architecture.md#security-architecture) - Security in system design
- [API Reference](./API-Reference.md#authentication) - API authentication details
- [Installation Guide](./Installation.md#permission-setup) - Setting up permissions
- [Development Guide](./Development.md#security-best-practices) - Secure coding practices
- [Technical Troubleshooting](./Troubleshooting-Technical.md#security-issues) - Security issue resolution

## Security Overview

The Quality Scorecard for Jira implements comprehensive security measures to protect data integrity, ensure user privacy, and maintain compliance with industry standards.

## Security Architecture

### Defense in Depth

```
┌─────────────────────────────────────────────────────┐
│                   User Layer                        │
│  ├── Authentication (Atlassian SSO)                 │
│  ├── Session Management                             │
│  └── RBAC (Role-Based Access Control)              │
├─────────────────────────────────────────────────────┤
│                Application Layer                    │
│  ├── Input Validation & Sanitization               │
│  ├── Output Encoding                               │
│  ├── Rate Limiting                                 │
│  └── Security Headers (CSP, HSTS)                  │
├─────────────────────────────────────────────────────┤
│                   API Layer                         │
│  ├── API Authentication (Forge Context)            │
│  ├── Request Validation                            │
│  ├── Response Filtering                            │
│  └── Audit Logging                                 │
├─────────────────────────────────────────────────────┤
│                 Storage Layer                       │
│  ├── Encryption at Rest                            │
│  ├── Access Control                                │
│  ├── Data Isolation                                │
│  └── Backup & Recovery                             │
└─────────────────────────────────────────────────────┘
```

## Authentication & Authorization

### Authentication Flow

```javascript
// Forge handles authentication automatically
// User context is provided in resolver
resolver.define('secureFunction', async (req) => {
  // User is already authenticated by Forge
  const { accountId, displayName } = req.context;
  
  // Additional authorization checks
  const hasPermission = await checkUserPermission(accountId, 'action');
  if (!hasPermission) {
    throw new Error('Unauthorized');
  }
  
  // Proceed with authorized action
});
```

### Permission Model

| Role | View Analytics | Create Evaluations | Edit Evaluations | Admin Functions |
|------|---------------|-------------------|------------------|-----------------|
| Viewer | ✅ | ❌ | ❌ | ❌ |
| Evaluator | ✅ | ✅ | Own only | ❌ |
| Team Lead | ✅ | ✅ | Team's | ❌ |
| Administrator | ✅ | ✅ | ✅ | ✅ |

### API Authentication Scopes

```yaml
# manifest.yml
permissions:
  scopes:
    - read:jira-work      # Read Jira issues
    - write:jira-work     # Write to Jira
    - storage:app         # App storage access
    - read:user:jira      # Read user information
```

For complete API documentation, see [API Reference](./API-Reference.md).

## Input Validation

### Validation Implementation

```javascript
// src/validators/inputValidator.js

// JQL Injection Prevention
const validateJQL = (jql) => {
  const DANGEROUS_PATTERNS = [
    /--/,              // SQL comment
    /;/,               // Statement separator
    /union\s+select/i, // Union attack
    /<script>/i,       // XSS attempt
    /javascript:/i     // JavaScript protocol
  ];
  
  for (const pattern of DANGEROUS_PATTERNS) {
    if (pattern.test(jql)) {
      throw new Error('Invalid JQL query');
    }
  }
  
  // Length validation
  if (jql.length > 2000) {
    throw new Error('JQL query too long');
  }
  
  return jql;
};

// Prevent CSV Injection
const sanitizeCSVValue = (value) => {
  // Prefix dangerous characters
  if (/^[=+\-@]/.test(value)) {
    return `'${value}`;
  }
  return value;
};
```

### Input Validation Rules

| Input Type | Validation Rules | Max Length |
|------------|-----------------|------------|
| JQL Query | No injection patterns, alphanumeric + operators | 2000 |
| Project Key | Uppercase letters and numbers only | 10 |
| Account ID | Alphanumeric with colons and hyphens | 100 |
| Feedback Text | HTML entities encoded | 5000 |
| Score | Integer 0-10 | - |
| Storage Key | Alphanumeric with hyphens and colons | 200 |

## Data Protection

### Encryption

#### At Rest
- All data stored in Forge Storage is encrypted
- AES-256 encryption standard
- Managed by Atlassian infrastructure

#### In Transit
- All communications use HTTPS/TLS 1.3
- Certificate pinning for API calls
- No sensitive data in URLs

### Data Classification

| Data Type | Classification | Protection Level |
|-----------|---------------|------------------|
| Evaluation Scores | Internal | Standard |
| User Feedback | Internal | Standard |
| Account IDs | Confidential | High |
| Email Addresses | Confidential | High |
| API Keys | Secret | Maximum |
| Audit Logs | Internal | Standard |

### Data Retention

```javascript
// Data retention policies
const RETENTION_POLICIES = {
  evaluations: 'unlimited',      // Keep all evaluations
  audit_logs: 90,                // 90 days
  deleted_items: 30,             // 30 days soft delete
  exports: 7,                    // 7 days for temporary exports
  cache: 0.5                     // 12 hours for cached data
};
```

## Security Headers

### Content Security Policy (CSP)

```yaml
# manifest.yml
permissions:
  content:
    styles:
      - 'self'
    scripts:
      - 'self'
```

### Response Headers

```javascript
// Security headers applied by Forge
{
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
  'X-Content-Type-Options': 'nosniff',
  'X-Frame-Options': 'DENY',
  'X-XSS-Protection': '1; mode=block',
  'Referrer-Policy': 'strict-origin-when-cross-origin'
}
```

## Rate Limiting

### Implementation

```javascript
// src/middleware/rateLimiter.js
class RateLimiter {
  constructor(options = {}) {
    this.requests = new Map();
    this.limits = {
      perMinute: options.perMinute || 60,
      perHour: options.perHour || 1000,
      concurrent: options.concurrent || 10
    };
  }
  
  async checkLimit(userId) {
    const now = Date.now();
    const userRequests = this.requests.get(userId) || [];
    
    // Clean old requests
    const recentRequests = userRequests.filter(
      time => now - time < 60000
    );
    
    if (recentRequests.length >= this.limits.perMinute) {
      throw new Error('Rate limit exceeded');
    }
    
    recentRequests.push(now);
    this.requests.set(userId, recentRequests);
  }
}
```

### Client & Platform Controls

- Client-side concurrency limiter (default 8 inflight) reduces contention and rate-limit hits.
- In-flight request deduplication coalesces identical calls.
- Platform (Forge/Jira) rate limits still apply; exponential backoff is used on `429`.

For API performance details, see [API Reference](./API-Reference.md#performance-and-response-times).

## Audit Logging

### Audit Service (Not Currently Integrated)

**Note:** While an audit service (`auditService.js`) exists in the codebase, it is not currently integrated into the main application. The following events would be logged if the service were active:

```javascript
// Potential audit events (not currently active)
const AUDIT_EVENTS = {
  // Authentication
  LOGIN_SUCCESS: 'User logged in',
  LOGIN_FAILURE: 'Failed login attempt',
  
  // Data Operations
  EVALUATION_CREATED: 'Evaluation created',
  EVALUATION_MODIFIED: 'Evaluation modified',
  
  // Administrative
  SETTINGS_CHANGED: 'Settings modified',
  DATA_EXPORTED: 'Data exported',
  
  // Security
  RATE_LIMIT_EXCEEDED: 'Rate limit exceeded',
  VALIDATION_FAILURE: 'Input validation failed'
};
```

**Current Tracking:** The app currently tracks:
- Project access changes (stored in `allowed-projects-audit`)
- Evaluation history (all evaluations are preserved)

### Audit Log Format

```json
{
  "timestamp": "2025-01-07T10:30:00Z",
  "eventType": "EVALUATION_CREATED",
  "userId": "user-123",
  "userEmail": "user@example.com",
  "ipAddress": "192.168.1.1",
  "resource": "evaluation-456",
  "details": {
    "ticketId": "PROJ-789",
    "score": 8.5
  },
  "result": "SUCCESS"
}
```

## Vulnerability Management

### Security Scanning

```bash
# Regular security scans
npm audit                    # Check npm dependencies
npm audit fix               # Auto-fix vulnerabilities
forge lint                  # Forge security linting
```

### Dependency Management

```json
// package.json
{
  "scripts": {
    "security-check": "npm audit --audit-level=moderate",
    "update-deps": "npm update && npm audit fix"
  }
}
```

### Known Vulnerabilities Addressed

| Vulnerability | Status | Mitigation |
|--------------|--------|------------|
| JQL Injection | ✅ Fixed | Input validation |
| XSS | ✅ Fixed | Output encoding, CSP |
| CSV Injection | ✅ Fixed | Value prefixing |
| ReDoS | ✅ Fixed | Regex optimization |
| Prototype Pollution | ✅ Fixed | Object.create(null) |
| Sensitive Data Logging | ✅ Fixed | Removed console.logs |

## GDPR Compliance

### Data Subject Rights

```javascript
// src/services/gdprService.js

// Right to Access
async function exportUserData(userId) {
  const evaluations = await getEvaluationsByUser(userId);
  const auditLogs = await getAuditLogsByUser(userId);
  return {
    evaluations,
    auditLogs,
    exportDate: new Date().toISOString()
  };
}

// Right to Erasure
async function deleteUserData(userId) {
  await deleteEvaluations(userId);
  await deleteAuditLogs(userId);
  await deleteUserPreferences(userId);
  return { deleted: true, timestamp: new Date() };
}

// Right to Rectification
async function updateUserData(userId, updates) {
  await updateUserRecords(userId, updates);
  await logDataUpdate(userId, updates);
  return { updated: true };
}
```

### Privacy by Design

- Minimal data collection
- Purpose limitation
- Data minimization
- Accuracy maintenance
- Storage limitation
- Security by default

## Incident Response

### Security Incident Procedure

1. **Detection**
   - Automated alerts
   - User reports
   - Log monitoring

2. **Assessment**
   - Severity classification
   - Impact analysis
   - Root cause identification

3. **Containment**
   - Isolate affected systems
   - Prevent spread
   - Preserve evidence

4. **Eradication**
   - Remove threat
   - Patch vulnerabilities
   - Update security measures

5. **Recovery**
   - Restore services
   - Verify integrity
   - Monitor for recurrence

6. **Lessons Learned**
   - Document incident
   - Update procedures
   - Train team

### Contact Information

| Role | Contact | Escalation Time |
|------|---------|-----------------|
| Nexus Frontier Support | support@nexusfrontier.net | Immediate |

## Security Best Practices

### For Developers

```javascript
// ✅ DO: Validate all inputs
const safeInput = validateAndSanitize(userInput);

// ❌ DON'T: Trust user input
const query = `SELECT * FROM table WHERE id = ${userInput}`;

// ✅ DO: Use parameterized queries
const result = await api.search({ jql: sanitizedJQL });

// ❌ DON'T: Concatenate user input
const jql = "project = " + userProject;

// ✅ DO: Handle errors securely
catch (error) {
  logger.error('Operation failed', { context });
  return { error: 'An error occurred' };
}

// ❌ DON'T: Expose internal details
catch (error) {
  return { error: error.stack };
}
```

### For Administrators

1. **Regular Updates**
   - Apply security patches promptly
   - Update dependencies monthly
   - Review security advisories

2. **Access Control**
   - Regular permission audits
   - Remove unused accounts
   - Enforce least privilege

3. **Monitoring**
   - Review audit logs daily
   - Set up security alerts
   - Monitor unusual patterns

4. **Training**
   - Security awareness sessions
   - Phishing simulations
   - Incident response drills

## Security Checklist

### Deployment Security Checklist

- [ ] All dependencies updated
- [ ] Security scan passed
- [ ] CSP headers configured
- [ ] Rate limiting enabled
- [ ] Audit logging active
- [ ] Input validation tested
- [ ] Error messages sanitized
- [ ] Permissions verified
- [ ] Backup configured
- [ ] Monitoring enabled

### Quarterly Security Review

- [ ] Dependency audit
- [ ] Permission review
- [ ] Log analysis
- [ ] Penetration testing
- [ ] Security training
- [ ] Policy updates
- [ ] Incident review
- [ ] Compliance check

## Compliance

### Standards Compliance

- **GDPR**: Full compliance with data protection
- **SOC 2**: Type II compliance through Atlassian
- **ISO 27001**: Security management standards
- **OWASP Top 10**: Protection against common vulnerabilities

### Audit Reports

Reports available upon request:
- Security assessment reports
- Penetration testing results
- Compliance certificates
- Vulnerability scan reports

## Related Resources

- [Architecture Documentation](./Architecture.md) - System architecture
- [API Reference](./API-Reference.md) - Complete API documentation
- [Installation Guide](./Installation.md) - Deployment and configuration
- [Development Guide](./Development.md) - Development practices
- [Technical Troubleshooting](./Troubleshooting-Technical.md) - Issue resolution

---

*For security concerns or vulnerability reports, contact: support@nexusfrontier.net*

*Last Updated: January 2025*

