# API Reference

## Overview

The Quality Compass provides a comprehensive set of Forge resolver functions for interacting with evaluations, analytics, and Jira data. All API calls are made through the Forge Bridge using the `invoke` function.

For security implementation details, see [Security Documentation](./Security.md).  
For development setup and testing, see [Development Guide](./Development.md).  
For troubleshooting API issues, see [Technical Troubleshooting](./Troubleshooting-Technical.md).

## Base URL and Environment

### Base URL
All API calls are made through the Forge Bridge, which handles the base URL internally. The actual Jira REST API base URL is:
- **Production**: `https://your-domain.atlassian.net`
- **Development**: Uses Forge tunnel to proxy requests

### Environment
Forge handles base URLs and authentication context internally via `api.asUser()`/`api.asApp()`. The app does not require manual base URL configuration.

## Authentication

All API calls are authenticated through the Forge platform context. The app uses two authentication modes:

- **`api.asUser()`**: Operations performed as the current user
- **`api.asApp()`**: Operations performed with app permissions

For detailed security and authentication information, see [Security Documentation](./Security.md#authentication--authorization).

## Core API Functions

### makeApiRequest

Generic API request handler for Jira REST API calls.

```javascript
invoke('makeApiRequest', {
  url: string,           // Relative URL path (e.g., '/rest/api/3/issue/PROJ-123')
  method?: string,       // HTTP method (GET, POST, PUT, DELETE)
  headers?: object,      // Additional headers
  body?: any            // Request body for POST/PUT
})
```

**Example:**
```javascript
const issue = await invoke('makeApiRequest', {
  url: '/rest/api/3/issue/PROJ-123',
  method: 'GET'
});
```

**Response:**
```json
{
  "id": "10001",
  "key": "PROJ-123",
  "fields": {
    "summary": "Issue summary",
    "status": { "name": "In Progress" },
    "assignee": { "displayName": "John Doe" }
  }
}
```

### getIssue

Retrieve a specific Jira issue by ID or key.

```javascript
invoke('getIssue', {
  issueIdOrKey: string  // Issue ID or key (e.g., '10001' or 'PROJ-123')
})
```

**Example:**
```javascript
const issue = await invoke('getIssue', {
  issueIdOrKey: 'PROJ-123'
});
```

**Response:**
```json
{
  "id": "10001",
  "key": "PROJ-123",
  "fields": {
    "summary": "Issue summary",
    "description": "Issue description",
    "issuetype": { "name": "Bug" },
    "priority": { "name": "High" },
    "status": { "name": "In Progress" },
    "assignee": {
      "accountId": "5b10a2844c20165700ede21g",
      "displayName": "John Doe"
    }
  }
}
```

### searchIssues

Search for Jira issues using JQL.

```javascript
invoke('searchIssues', {
  jql: string,           // JQL query
  maxResults?: number,   // Maximum results (default: 50, max: 100)
  startAt?: number,      // Pagination offset (default: 0)
  fields?: string[]      // Fields to return
})
```

**Example:**
```javascript
const results = await invoke('searchIssues', {
  jql: 'project = PROJ AND status = "In Progress"',
  maxResults: 20,
  fields: ['summary', 'status', 'assignee']
});
```

**Response:**
```json
{
  "issues": [
    {
      "id": "10001",
      "key": "PROJ-123",
      "fields": {
        "summary": "Issue summary",
        "status": { "name": "In Progress" },
        "assignee": { "displayName": "John Doe" }
      }
    }
  ],
  "total": 45,
  "maxResults": 20,
  "startAt": 0
}
```

## Storage API Functions

### getStorageValue

Retrieve a value from Forge storage.

```javascript
invoke('getStorageValue', {
  key: string  // Storage key
})
```

**Example:**
```javascript
const evaluations = await invoke('getStorageValue', {
  key: 'evaluations'
});
```

**Common Storage Keys (actual usage):**
- `ticket-evaluations` - All evaluations
- `project-evaluations-<PROJECT_KEY>` - Project-scoped evaluations
- `evaluation-criteria` - Criteria configuration (internal)
- `evaluator-name-mapping` - Manual evaluator name overrides
- `evaluator-name-removed` - Removed evaluator IDs list (internal)
- `allowed-projects` - Allowed project keys
- `allowed-projects-audit` - Project access audit entries
- `coverage-targets` - Coverage target settings

### setStorageValue

Store a value in Forge storage.

```javascript
invoke('setStorageValue', {
  key: string,    // Storage key
  value: any      // Value to store
})
```

**Example:**
```javascript
await invoke('setStorageValue', {
  key: 'evaluation-criteria',
  value: {
    accuracy: { name: 'Accuracy', weight: 1.0 },
    completeness: { name: 'Completeness', weight: 1.0 }
  }
});
```

**Response:**
```json
{
  "success": true
}
```

### deleteStorageValue

Delete a value from Forge storage.

```javascript
invoke('deleteStorageValue', {
  key: string  // Storage key to delete
})
```

**Example:**
```javascript
await invoke('deleteStorageValue', {
  key: 'user-selected-project:user123'
});
```

## User Management API

### getUser

Get user information by account ID.

```javascript
invoke('getUser', {
  accountId: string  // Atlassian account ID
})
```

**Example:**
```javascript
const user = await invoke('getUser', {
  accountId: '5b10a2844c20165700ede21g'
});
```

**Response:**
```json
{
  "accountId": "5b10a2844c20165700ede21g",
  "displayName": "John Doe",
  "emailAddress": "john.doe@example.com",
  "avatarUrls": {
    "48x48": "https://avatar-url.com/48x48.png"
  },
  "active": true
}
```

### batchGetUsers

Get multiple users in a single request.

```javascript
invoke('batchGetUsers', {
  accountIds: string[]  // Array of account IDs
})
```

**Example:**
```javascript
const users = await invoke('batchGetUsers', {
  accountIds: ['5b10a2844c20165700ede21g', '5b10a2844c20165700ede22h']
});
```

**Response:**
```json
{
  "5b10a2844c20165700ede21g": {
    "displayName": "John Doe",
    "emailAddress": "john.doe@example.com"
  },
  "5b10a2844c20165700ede22h": {
    "displayName": "Jane Smith",
    "emailAddress": "jane.smith@example.com"
  }
}
```

## Evaluation Management

### setEvaluatorName

Set a manual override for an evaluator's display name.

```javascript
invoke('setEvaluatorName', {
  evaluatorId: string,  // Evaluator's account ID
  displayName: string   // Display name override
})
```

**Example:**
```javascript
await invoke('setEvaluatorName', {
  evaluatorId: '5b10a2844c20165700ede21g',
  displayName: 'John Doe (QA Lead)'
});
```

### removeEvaluatorName

Remove a manual override for an evaluator's display name.

```javascript
invoke('removeEvaluatorName', {
  evaluatorId: string
})
```

### getEvaluatorNameMapping

Retrieve the current evaluator name mapping.

```javascript
invoke('getEvaluatorNameMapping', {})
```

## Data Models

### TicketEvaluation

```typescript
interface TicketEvaluation {
  ticketId: string;
  ticketKey: string;
  evaluatorId: string;
  evaluatorDisplayName?: string;
  averageScore: number;
  criteria: Record<string, number>;  // criteriaId -> score (0-10)
  feedback?: string;
  timestamp: string;  // ISO 8601 format
  projectKey?: string;
  priority?: string;
  issueType?: string;
  status?: string;
  assigneeId?: string;
  assigneeDisplayName?: string;
}
```

### EvaluationCriteria

```typescript
interface EvaluationCriteria {
  [key: string]: {
    name: string;
    description?: string;
    weight: number;  // Default: 1.0
    enabled: boolean;
  }
}
```

### StorageStructure

```typescript
interface StorageStructure {
  evaluations: TicketEvaluation[];
  'evaluation-criteria': EvaluationCriteria;
  'allowed-projects': string[];
  'evaluator-name-mapping': Record<string, string>;
  'audit-logs': AuditLogEntry[];
}
```

## Error Handling

All API calls may return errors in the following format:

```json
{
  "error": true,
  "message": "Error description",
  "code": "ERROR_CODE",
  "details": {}
}
```

### Common Error Codes

| Code | Description | Resolution | HTTP Status |
|------|-------------|------------|-------------|
| `UNAUTHORIZED` | Missing permissions | Check user permissions | 401 |
| `FORBIDDEN` | Access denied | Verify project access | 403 |
| `NOT_FOUND` | Resource not found | Verify resource exists | 404 |
| `VALIDATION_ERROR` | Invalid input | Check input parameters | 400 |
| `RATE_LIMIT` | Rate limit exceeded | Wait before retrying | 429 |
| `STORAGE_ERROR` | Storage operation failed | Check storage quota | 500 |
| `STORAGE_QUOTA_EXCEEDED` | Storage limit reached | Clean up old data | 507 |
| `NETWORK_ERROR` | Network request failed | Check connectivity | 503 |
| `TIMEOUT` | Request timed out | Retry with smaller dataset | 504 |
| `CONFLICT` | Resource conflict | Check for duplicate operations | 409 |
| `INVALID_JQL` | JQL syntax error | Validate JQL query | 400 |
| `SESSION_EXPIRED` | Authentication expired | Re-authenticate user | 401 |

### Error Handling Example

```javascript
try {
  const result = await invoke('getIssue', {
    issueIdOrKey: 'PROJ-123'
  });
  // Handle success
} catch (error) {
  if (error.code === 'NOT_FOUND') {
    console.error('Issue not found');
  } else if (error.code === 'UNAUTHORIZED') {
    console.error('Permission denied');
  } else {
    console.error('Unexpected error:', error.message);
  }
}
```

## Performance and Response Times

### Expected Response Times

| Operation Type | Expected Time | Maximum Time | Notes |
|---------------|---------------|--------------|-------|
| Single Issue Fetch | 100-300ms | 1s | Depends on issue complexity |
| JQL Search (50 items) | 200-500ms | 2s | Varies with query complexity |
| Batch User Fetch | 150-400ms | 1.5s | Up to 100 users |
| Storage Read | 50-150ms | 500ms | Cached data faster |
| Storage Write | 100-300ms | 1s | Depends on data size |
| Analytics Calculation | 500-2000ms | 5s | Large datasets may take longer |
| Export Generation | 1-5s | 30s | Depends on data volume |

### Performance Optimization Tips
- Use batch operations when possible
- Implement client-side caching
- Paginate large result sets
- Use field filtering to reduce payload size

For performance troubleshooting, see [Technical Troubleshooting](./Troubleshooting-Technical.md#performance-issues).

## Rate Limiting

No custom application rate limiting is implemented. Forge/Jira platform limits apply to API requests. Handle `429` responses with retries and backoff.

For guidance, see [Security Documentation](./Security.md#rate-limiting).

## Pagination

For endpoints returning large datasets:

```javascript
// Pagination parameters
{
  maxResults: 50,  // Items per page (max: 100)
  startAt: 0       // Starting index
}

// Response includes
{
  total: 500,      // Total items available
  maxResults: 50,  // Items in this response
  startAt: 0,      // Current offset
  isLast: false    // Is this the last page
}
```

### Pagination Example

```javascript
async function getAllIssues(jql) {
  const allIssues = [];
  let startAt = 0;
  let isLast = false;

  while (!isLast) {
    const response = await invoke('searchIssues', {
      jql,
      maxResults: 50,
      startAt
    });
    
    allIssues.push(...response.issues);
    startAt += response.maxResults;
    isLast = response.startAt + response.maxResults >= response.total;
  }

  return allIssues;
}
```

## Webhooks (Future)

*Note: Webhook support is planned for future releases.*

```javascript
// Planned webhook events
{
  'evaluation.created': {},
  'evaluation.updated': {},
  'evaluation.deleted': {},
  'criteria.changed': {},
  'threshold.exceeded': {}
}
```

## Best Practices

### 1. Error Handling
Always wrap API calls in try-catch blocks and handle specific error codes.

### 2. Caching
Implement client-side caching for frequently accessed data:
```javascript
const cacheKey = `issue_${issueKey}`;
const cached = cache.get(cacheKey);
if (cached) return cached;

const data = await invoke('getIssue', { issueIdOrKey: issueKey });
cache.set(cacheKey, data, { ttl: 300000 }); // 5 minutes
return data;
```

### 3. Batch Operations
Use batch endpoints when fetching multiple items:
```javascript
// Good - Single batch request
const users = await invoke('batchGetUsers', { accountIds });

// Avoid - Multiple individual requests
for (const id of accountIds) {
  const user = await invoke('getUser', { accountId: id });
}
```

### 4. Input Validation
Validate inputs before making API calls:
```javascript
function validateJQL(jql) {
  if (!jql || jql.length > 2000) {
    throw new Error('Invalid JQL query');
  }
  // Additional validation...
}
```

### 5. Rate Limit Handling
Implement exponential backoff for rate limit errors:
```javascript
async function withRetry(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.code === 'RATE_LIMIT' && i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
      } else {
        throw error;
      }
    }
  }
}
```

## Additional API Functions

### getFilteredEvaluations

Retrieve evaluations with specific filters.

```javascript
invoke('getFilteredEvaluations', {
  projectKey?: string,
  evaluatorId?: string,
  dateFrom?: string,  // ISO 8601 format
  dateTo?: string,    // ISO 8601 format
  minScore?: number,
  maxScore?: number
})
```

**Response Time:** 200-800ms depending on filter complexity

### getTicketMetadataBatch

Fetch metadata for multiple tickets efficiently.

```javascript
invoke('getTicketMetadataBatch', {
  ticketIds: string[],  // Array of ticket IDs
  fields?: string[]      // Optional field filter
})
```

**Response Time:** 300-1500ms for up to 50 tickets

### getCoverageTargets

Retrieve configured coverage targets.

```javascript
invoke('getCoverageTargets', {})
```

**Response Time:** 50-150ms

### setCoverageTargets

Update coverage targets (admin only).

```javascript
invoke('setCoverageTargets', {
  overall: number,      // 0-100
  byPriority: {        // Optional priority-specific targets
    Highest: number,
    High: number,
    Medium: number,
    Low: number,
    Lowest: number
  }
})
```

**Response Time:** 100-300ms

## Testing

### Test Environment
Use Forge tunnel for local development:
```bash
forge tunnel
```

For complete development setup, see [Development Guide](./Development.md#development-environment-setup).

### Sample Test Data
```javascript
// Test evaluation creation
const testEvaluation = {
  ticketId: '10001',
  ticketKey: 'TEST-1',
  evaluatorId: 'test-user-id',
  averageScore: 8.5,
  criteria: {
    accuracy: 9,
    completeness: 8,
    communication: 8.5
  },
  feedback: 'Test feedback',
  timestamp: new Date().toISOString()
};
```

## Related Documentation

- [Architecture Overview](./Architecture.md) - System design and components
- [Installation Guide](./Installation.md) - Setup and configuration
- [Development Guide](./Development.md) - Development environment and practices
- [Security Documentation](./Security.md) - Security implementation details
- [Technical Troubleshooting](./Troubleshooting-Technical.md) - Solving technical issues

---

*Last Updated: January 2025*

