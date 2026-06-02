# Installation Guide

## Quick Links
- [Architecture Overview](./Architecture.md) - Understand the system before installation
- [API Reference](./API-Reference.md) - API documentation for integration
- [Development Guide](./Development.md) - For development setup
- [Security Documentation](./Security.md) - Security considerations
- [Technical Troubleshooting](./Troubleshooting-Technical.md) - Installation issues

## Prerequisites

Before installing the Quality Scorecard for Jira, ensure you have the following:

### System Requirements
- **Jira Cloud**: Active Jira Cloud instance
- **Permissions**: Jira administrator access
- **Browser**: Modern web browser (Chrome, Firefox, Safari, Edge)
- **Forge CLI**: Version 6.x or higher (for development)

### Required Jira Permissions
The installing user must have:
- Jira Administrator permissions
- Ability to install Forge apps
- Project administration rights for target projects

## Installation Steps

### Step 1: Install via Forge CLI (recommended for current build)

1. **Register and Deploy**
   ```bash
   forge register
   npm run build
   forge deploy --environment development
   ```

2. **Install into your Jira site**
   ```bash
   forge install --environment development --site <your-site>.atlassian.net --product jira
   ```

3. **Verify Installation**
   - Open your Jira project → Apps → Quality Scorecard for Jira
   - Ensure dashboards/tabs load

### Step 2: Initial Configuration

#### Configure Allowed Projects

1. **Access App Settings**
   ```
   Navigate to: Project Settings → Apps → Quality Scorecard for Jira
   ```

2. **Set Allowed Projects**
   ```javascript
   // The app will automatically detect and use the current project
   // Additional projects can be configured via the admin interface
   ```

3. **Configure Evaluation Criteria**
   - Access the Admin View
   - Navigate to Settings → Evaluation Criteria
   - Define your evaluation criteria (default criteria are pre-configured)

#### Default Evaluation Criteria
```json
{
  "accuracy": {
    "name": "Accuracy",
    "description": "Correctness of the solution",
    "weight": 1.0,
    "enabled": true
  },
  "completeness": {
    "name": "Completeness", 
    "description": "All requirements addressed",
    "weight": 1.0,
    "enabled": true
  },
  "communication": {
    "name": "Communication",
    "description": "Clarity of communication",
    "weight": 1.0,
    "enabled": true
  }
}
```

### Step 3: Permission Setup

#### Configure User Roles

1. **Evaluators**
   - Must have "Browse Projects" permission
   - Require "Edit Issues" permission for evaluation
   
2. **Viewers**
   - Need "Browse Projects" permission
   - Can view analytics without edit rights

3. **Administrators**
   - Require "Administer Projects" permission
   - Full access to all features

#### Setting Permissions via Jira
```
Project Settings → Permissions → Permission Schemes
- Browse Projects: All users
- Edit Issues: Evaluators and above
- Administer Projects: Administrators only
```

For detailed permission model, see [Security Documentation](./Security.md#authentication--authorization).

### Step 4: Environment Configuration

#### Storage Configuration
The app automatically configures storage with these defaults:

```javascript
// Automatic storage configuration
{
  "storage": {
    "type": "forge",
    "encryption": "at-rest",
    "retention": "unlimited",
    "backup": "automatic"
  }
}
```

#### API Rate Limits
The app uses a client-side concurrency limiter (default 8) and in-flight request deduplication to smooth bursts. Platform limits still apply.

### Step 5: Verification

#### Verify Installation Success

1. **Check App Access**
   ```
   Navigate to any project → Apps → Quality Scorecard for Jira
   The dashboard should load without errors
   ```

2. **Test Basic Functionality**
   - Create a test evaluation
   - View analytics dashboard
   - Export sample data

3. **Verify Permissions**
   - Test with different user roles
   - Confirm appropriate access levels

#### Health Check Commands
For developers with Forge CLI:
```bash
# Check app status
forge list

# View app logs
forge logs

# Check deployment
forge install list
```

## Advanced Configuration

### Custom Domain Configuration
If using a custom domain:

```yaml
# manifest.yml adjustment
app:
  runtime:
    name: nodejs18.x
  
permissions:
  external:
    fetch:
      backend:
        - 'https://your-domain.com'
```

### Proxy Configuration
For environments requiring proxy:

```javascript
// Set in environment variables
HTTPS_PROXY=http://proxy.company.com:8080
HTTP_PROXY=http://proxy.company.com:8080
```

### Performance Tuning

#### Cache Configuration
```javascript
// Adjust cache TTL values in cacheService.ts
export const CACHE_TTL = {
  SHORT: 5 * 60 * 1000,    // 5 minutes
  MEDIUM: 15 * 60 * 1000,  // 15 minutes
  LONG: 30 * 60 * 1000,    // 30 minutes
  EXTRA_LONG: 60 * 60 * 1000 // 1 hour
};
```

#### Batch Processing Limits
```javascript
// Adjust in src/index.js
const BATCH_SIZE = 50; // Number of items per batch
const MAX_CONCURRENT = 5; // Concurrent API calls
```

## Troubleshooting Installation

### Common Issues and Solutions

#### Issue: App Not Appearing in Project
**Solution:**
1. Verify installation completed successfully
2. Check project permissions
3. Refresh browser cache
4. Re-install if necessary

#### Issue: Permission Denied Errors
**Solution:**
```bash
# Verify permissions in manifest.yml
permissions:
  scopes:
    - read:jira-work
    - write:jira-work
    - storage:app
    - read:user:jira
```

For more troubleshooting steps, see [Technical Troubleshooting](./Troubleshooting-Technical.md#installation-issues).

#### Issue: Storage Errors
**Solution:**
1. Check Forge storage quota
2. Verify storage permissions
3. Clear corrupted data:
```javascript
// Emergency storage reset (use with caution)
await storage.delete('evaluations');
await storage.set('evaluations', []);
```

#### Issue: Performance Problems
**Solution:**
1. Check network connectivity
2. Verify Jira instance performance
3. Adjust cache settings
4. Reduce batch sizes

### Installation Validation Checklist

- [ ] App appears in Manage Apps
- [ ] Can access from project navigation
- [ ] Admin dashboard loads correctly
- [ ] Can create evaluations
- [ ] Analytics display properly
- [ ] Export functionality works
- [ ] All user roles can access appropriately
- [ ] No console errors in browser
- [ ] API calls succeed
- [ ] Storage operations work

## Uninstallation

### To Remove the App

1. **Navigate to Manage Apps**
   ```
   Apps → Manage apps → Quality Scorecard for Jira
   ```

2. **Uninstall**
   - Click the app name
   - Select "Uninstall"
   - Confirm removal

3. **Data Cleanup**
   - Evaluation data remains in Forge storage after uninstall
   - Export all data before uninstalling
   - Contact Atlassian support if data removal is required

### Data Export Before Uninstall
```javascript
// Export all data before removal
1. Go to Analytics Dashboard
2. Select "Export All Data"
3. Choose Excel format
4. Save the file securely
```

## Migration from Previous Versions

### Version 0.x to 1.0

1. **Backup Existing Data**
   - Export all evaluations
   - Save configuration settings

2. **Update Process**
   ```bash
   # Via Forge CLI
   forge deploy --environment production
   ```

3. **Data Migration**
   - Automatic migration runs on first launch
   - Verify data integrity post-migration

### Configuration Migration
```javascript
// Old format
{
  "criteria": ["accuracy", "completeness"]
}

// New format (automatic conversion)
{
  "criteria": {
    "accuracy": { "name": "Accuracy", "weight": 1.0 },
    "completeness": { "name": "Completeness", "weight": 1.0 }
  }
}
```

## Security Considerations

### Post-Installation Security

1. **Review Permissions**
   - Audit user access levels
   - Remove unnecessary permissions
   - Enable audit logging

2. **Configure Security Settings**
   ```javascript
   // Enable strict validation
   ENABLE_STRICT_VALIDATION=true
   
   // Set session timeout
   SESSION_TIMEOUT_MINUTES=30
   ```

3. **Monitor Access**
   - Review audit logs regularly
   - Set up alerts for suspicious activity
   - Monitor API usage

## Support and Resources

### Getting Help

- **Documentation**: [Full Documentation](../README.md)
- **API Reference**: [API Documentation](./API-Reference.md)
- **Development Setup**: [Development Guide](./Development.md)
- **Security**: [Security Documentation](./Security.md)
- **Troubleshooting**: [Technical Troubleshooting](./Troubleshooting-Technical.md)
- **Technical Support**: support@nexusfrontier.net

### Useful Commands

```bash
# Development installation
forge deploy

# Production installation
forge deploy --environment production

# View logs
forge logs --environment production

# Check installation
forge install list

# Tunnel for development
forge tunnel
```

---

*Last Updated: January 2025*

