# Technical Troubleshooting Guide

## Overview

This guide provides solutions for technical issues that administrators and developers may encounter with the Quality Compass.

## Quick Links
- [Architecture Overview](./Architecture.md) - System design for context
- [API Reference](./API-Reference.md#error-handling) - API error codes and handling
- [Installation Guide](./Installation.md#troubleshooting-installation) - Installation issues
- [Development Guide](./Development.md#debugging) - Development debugging
- [Security Documentation](./Security.md#incident-response) - Security incidents

## Common Technical Issues

### Installation Issues

#### App Not Appearing After Installation

**Symptoms:**
- App installed successfully but not visible in projects
- "Apps" menu doesn't show Quality Compass

**Diagnosis:**
```bash
# Check installation status
forge install list

# Verify app deployment
forge environment production
```

**Solutions:**
1. **Clear Jira cache:**
   ```
   Settings → System → Cache Management → Clear all
   ```

2. **Verify manifest configuration:**
   ```yaml
   # manifest.yml should include
   modules:
     jira:projectPage:
       - key: qa-app-project-page
         resource: main
   ```

3. **Reinstall the app:**
   ```bash
   forge uninstall
   forge deploy --environment production
   forge install
   ```

#### Permission Errors During Installation

**Error:** `Insufficient permissions to install app`

**Solution:**
```bash
# Verify admin permissions
# User must be Jira administrator

# Check Forge permissions
forge whoami

# Re-authenticate if needed
forge logout
forge login
```

For complete installation instructions, see [Installation Guide](./Installation.md).

---

### Storage Issues

#### Storage Quota Exceeded

**Error:** `ForgeStorageError: Storage quota exceeded`

**Diagnosis:**
```javascript
// Check storage usage
const storageInfo = await storage.query()
  .where('key', startsWith('evaluations'))
  .limit(1000)
  .getMany();
console.log(`Items in storage: ${storageInfo.results.length}`);
```

**Solutions:**

1. **Clean old data:**
```javascript
// Remove evaluations older than 1 year
const oneYearAgo = new Date();
oneYearAgo.setFullYear(oneYearAgo.getFullYear() - 1);

const oldEvaluations = evaluations.filter(
  e => new Date(e.timestamp) < oneYearAgo
);
// Archive before deletion
await exportToBackup(oldEvaluations);
// Remove from storage
await storage.set('evaluations', currentEvaluations);
```

2. **Optimize storage structure:**
```javascript
// Split large arrays into chunks
const CHUNK_SIZE = 100;
for (let i = 0; i < evaluations.length; i += CHUNK_SIZE) {
  const chunk = evaluations.slice(i, i + CHUNK_SIZE);
  await storage.set(`evaluations_${i / CHUNK_SIZE}`, chunk);
}
```

#### Storage Read/Write Failures

**Error:** `Failed to read/write storage`

**Solutions:**
```javascript
// Implement retry logic
async function storageWithRetry(operation, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await operation();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
}

// Usage
const data = await storageWithRetry(() => 
  storage.get('evaluations')
);
```

---

### Performance Issues

#### Slow Dashboard Loading

**Symptoms:**
- Analytics dashboard takes > 10 seconds to load
- Timeout errors

**Diagnosis:**
```javascript
// Add performance logging
console.time('DataFetch');
const data = await fetchAllEvaluations();
console.timeEnd('DataFetch');

console.time('Processing');
const processed = processAnalytics(data);
console.timeEnd('Processing');
```

**Solutions:**

1. **Persistent cache + SWR:** Advanced analytics reads from IndexedDB first and refreshes in background.

2. **Implement pagination:**
```javascript
// Instead of loading all data
const allEvaluations = await storage.get('evaluations');

// Load in pages
async function* getEvaluationsPaginated(pageSize = 100) {
  let page = 0;
  while (true) {
    const data = await storage.get(`evaluations_page_${page}`);
    if (!data || data.length === 0) break;
    yield data;
    page++;
  }
}
```

3. **Add caching layer:**
```javascript
// Cache processed analytics
const cacheKey = `analytics_${dateRange}_${filters}`;
const cached = await storage.get(cacheKey);

if (cached && cached.timestamp > Date.now() - 3600000) {
  return cached.data;
}

const fresh = await calculateAnalytics();
await storage.set(cacheKey, {
  data: fresh,
  timestamp: Date.now()
});
```

#### Memory Leaks

**Symptoms:**
- Browser tab crashes
- Increasing memory usage over time

**Diagnosis:**
```javascript
// Monitor memory usage
if (performance.memory) {
  console.log({
    used: performance.memory.usedJSHeapSize / 1048576,
    total: performance.memory.totalJSHeapSize / 1048576,
    limit: performance.memory.jsHeapSizeLimit / 1048576
  });
}
```

**Solutions:**
```javascript
// Clean up event listeners
useEffect(() => {
  const handler = (e) => handleEvent(e);
  window.addEventListener('resize', handler);
  
  return () => {
    window.removeEventListener('resize', handler);
  };
}, []);

// Clear large data structures
useEffect(() => {
  return () => {
    // Cleanup on unmount
    largeDataArray = null;
    cacheMap.clear();
  };
}, []);
```

---

### API Issues

#### Rate Limiting Errors

**Error:** `429 Too Many Requests`

**Solutions:**

1. **Implement exponential backoff:**
```javascript
async function apiCallWithBackoff(fn, maxRetries = 5) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status !== 429 || i === maxRetries - 1) {
        throw error;
      }
      const delay = Math.min(1000 * Math.pow(2, i), 30000);
      await new Promise(r => setTimeout(r, delay));
    }
  }
}
```

2. **Batch API requests:**
```javascript
// Instead of individual calls
for (const id of userIds) {
  const user = await getUser(id);
}

// Use batch endpoint
const users = await batchGetUsers(userIds);
```

#### Authentication Failures

**Error:** `401 Unauthorized`

**Diagnosis:**
```javascript
// Check authentication context
resolver.define('debugAuth', async (req) => {
  console.log('Context:', req.context);
  console.log('Principal:', req.principal);
  return { authenticated: !!req.context.accountId };
});
```

**Solutions:**
```yaml
# Verify permissions in manifest.yml
permissions:
  scopes:
    - read:jira-work
    - write:jira-work
    - storage:app
    - read:user:jira
```

For authentication details, see [Security Documentation](./Security.md#authentication--authorization) and [API Reference](./API-Reference.md#authentication).

---

### Build and Deployment Issues

#### Build Failures

**Error:** `Build failed: Module not found`

**Solutions:**
```bash
# Clean install dependencies
rm -rf node_modules package-lock.json
npm install

# Clear build cache
rm -rf static/hello-world/dist
npm run build

# Check for version conflicts
npm ls
```

#### Worker Build Errors (Vite)

**Error:** `vite:worker-import-meta-url Invalid value 'iife' for option 'worker-format'`

**Solution:**
```ts
// Create workers via URL with module type
new Worker(new URL('./workers/advancedAnalyticsWorker.ts', import.meta.url), { type: 'module' });

// Ensure vite.config.ts includes
export default defineConfig({
  worker: { format: 'es' }
});
```

#### Deployment Failures

**Error:** `Deployment failed: Invalid manifest`

**Validation:**
```bash
# Validate manifest
forge lint

# Check manifest syntax
cat manifest.yml | yaml-lint
```

**Common manifest issues:**
```yaml
# ❌ Wrong
app:
  id: ${APP_ID}  # Variable not set

# ✅ Correct
app:
  id: ari:cloud:ecosystem::app/actual-id-here
```

---

### Database/Query Issues

#### JQL Query Failures

**Error:** `Invalid JQL query`

**Debugging:**
```javascript
// Log the actual JQL being sent
console.log('JQL Query:', jql);

// Test in Jira directly
// Navigate to Issues → Search → Advanced
// Paste the JQL query
```

**Common JQL issues:**
```javascript
// ❌ Wrong: Unescaped special characters
const jql = `project = ${projectKey} AND summary ~ "${searchTerm}"`;

// ✅ Correct: Properly escaped
const jql = `project = ${projectKey} AND summary ~ "${searchTerm.replace(/"/g, '\\"')}"`;
```

#### Data Corruption

**Symptoms:**
- Inconsistent evaluation data
- Missing or duplicate entries

**Recovery procedure:**
```javascript
// 1. Export current data
const backup = await storage.get('evaluations');
await exportToFile(backup, 'backup.json');

// 2. Validate data structure
const validated = backup.filter(e => {
  return e.ticketId && 
         e.evaluatorId && 
         typeof e.averageScore === 'number' &&
         e.timestamp;
});

// 3. Remove duplicates
const unique = Array.from(
  new Map(validated.map(e => 
    [`${e.ticketId}_${e.timestamp}`, e]
  )).values()
);

// 4. Restore clean data
await storage.set('evaluations', unique);
```

---

### Frontend Issues

#### React Component Errors

**Error:** `Cannot read property of undefined`

**Debugging:**
```javascript
// Add error boundaries
class ErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    console.error('Component error:', error, errorInfo);
    // Log to monitoring service
  }
  
  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

**Prevention:**
```javascript
// Use optional chaining
const score = evaluation?.criteria?.accuracy ?? 0;

// Provide default values
const { 
  data = [], 
  loading = false, 
  error = null 
} = useAnalytics();
```

#### State Management Issues

**Symptoms:**
- State not updating
- Stale data displayed

**Solutions:**
```javascript
// Use functional updates for state
setState(prevState => ({
  ...prevState,
  newValue: calculated
}));

// Force re-render if needed
const [, forceUpdate] = useReducer(x => x + 1, 0);
// Call forceUpdate() when needed

// Clear stale cache
useEffect(() => {
  cacheService.clear();
}, [filters]);
```

---

### Security Issues

#### CSP Violations

**Error:** `Refused to execute inline script`

**Solution:**
```javascript
// Move inline scripts to external files
// ❌ Wrong
<button onclick="handleClick()">

// ✅ Correct
<button id="myButton">
<script src="handlers.js"></script>

// In handlers.js
document.getElementById('myButton')
  .addEventListener('click', handleClick);
```

#### XSS Prevention Failures

**Diagnosis:**
```javascript
// Test for XSS vulnerabilities
const testInputs = [
  '<script>alert("XSS")</script>',
  'javascript:alert("XSS")',
  '<img src=x onerror=alert("XSS")>'
];

testInputs.forEach(input => {
  const sanitized = sanitizeInput(input);
  console.assert(!sanitized.includes('<script>'));
});
```

For complete security implementation, see [Security Documentation](./Security.md#input-validation).

---

## Debugging Tools

### Forge CLI Commands

```bash
# View logs
forge logs --environment production

# Check app status
forge list

# Validate configuration
forge lint

# Run in development mode
forge tunnel
```

### Browser DevTools

```javascript
// Performance profiling
performance.mark('myTask-start');
// ... code to profile ...
performance.mark('myTask-end');
performance.measure('myTask', 'myTask-start', 'myTask-end');

// Memory profiling
console.memory

// Network monitoring
// Use Network tab to check API calls
```

### Logging Strategy

```javascript
// Structured logging
const log = (level, message, context = {}) => {
  const entry = {
    timestamp: new Date().toISOString(),
    level,
    message,
    ...context
  };
  
  if (process.env.NODE_ENV === 'development') {
    console.log(entry);
  } else {
    // Send to logging service
    sendToLoggingService(entry);
  }
};

// Usage
log('error', 'API call failed', {
  endpoint: '/api/evaluate',
  error: error.message,
  userId: context.accountId
});
```

---

## Emergency Procedures

### System Down

1. **Immediate Actions:**
   ```bash
   # Check deployment status
   forge install list
   
   # Check for Atlassian incidents
   # Visit: https://status.atlassian.com
   ```

2. **Rollback if needed:**
   ```bash
   # Deploy previous version
   git checkout <previous-version-tag>
   forge deploy --environment production
   ```

### Data Loss

1. **Check backups:**
   ```javascript
   // Automated backups location
   const backups = await storage.get('backups');
   ```

2. **Restore from export:**
   ```javascript
   // If users exported data
   const importData = JSON.parse(fileContent);
   await storage.set('evaluations', importData.evaluations);
   ```

### Security Breach

1. **Immediate response:**
   - Disable affected features
   - Revoke compromised credentials
   - Enable additional logging

2. **Investigation:**
   ```javascript
   // Check audit logs
   const suspiciousActivity = auditLogs.filter(log => 
     log.eventType === 'UNAUTHORIZED_ACCESS' ||
     log.result === 'FAILURE'
   );
   ```

---

## Monitoring Setup

### Health Checks

```javascript
// Endpoint for monitoring
resolver.define('healthCheck', async () => {
  const checks = {
    storage: false,
    api: false,
    timestamp: new Date().toISOString()
  };
  
  try {
    await storage.get('health-check');
    checks.storage = true;
  } catch (e) {
    console.error('Storage check failed:', e);
  }
  
  try {
    await api.asApp().requestJira('/rest/api/3/myself');
    checks.api = true;
  } catch (e) {
    console.error('API check failed:', e);
  }
  
  return checks;
});
```

### Alerts Configuration

```javascript
// Alert thresholds
const ALERT_THRESHOLDS = {
  errorRate: 0.05,        // 5% error rate
  responseTime: 5000,     // 5 seconds
  storageUsage: 0.9,      // 90% of quota
  memoryUsage: 0.8        // 80% of limit
};
```

---

## Support Escalation

### Escalation Path

1. **Level 1: Check documentation**
   - This troubleshooting guide
   - FAQ section
   - Known issues list

2. **Level 2: Development team**
   - Email: dev-team@organization.com
   - Slack: #quality-compass-dev

3. **Level 3: Atlassian support**
   - For Forge platform issues
   - https://support.atlassian.com

### Information to Provide

When reporting issues, include:
- Error messages (exact text)
- Browser console output
- Network tab HAR file
- Steps to reproduce
- Environment details
- Time of occurrence
- Affected users

## Additional Resources

- [Architecture Overview](./Architecture.md) - System architecture
- [API Reference](./API-Reference.md) - API documentation
- [Installation Guide](./Installation.md) - Setup instructions
- [Development Guide](./Development.md) - Development workflow
- [Security Documentation](./Security.md) - Security measures

---

*For urgent technical issues, contact: tech-support@your-organization.com*

*Last Updated: January 2025*

