# Development Guide

## Related Documentation
- [Architecture Overview](./Architecture.md) - System architecture and design
- [API Reference](./API-Reference.md) - Complete API documentation
- [Installation Guide](./Installation.md) - Production deployment
- [Security Documentation](./Security.md) - Security best practices
- [Technical Troubleshooting](./Troubleshooting-Technical.md) - Debugging and issues

## Development Environment Setup

### Prerequisites

Before starting development, ensure you have:

```bash
# Required tools
node --version  # v18.x or higher
npm --version   # v8.x or higher
forge --version # v6.x or higher

# Atlassian account
# Jira Cloud development instance
```

### Initial Setup

#### 1. Clone the Repository
```bash
git clone https://github.com/your-org/quality-compass.git
cd quality-compass
```

#### 2. Install Dependencies
```bash
# Install Forge CLI globally
npm install -g @forge/cli

# Install project dependencies
npm install

# Install frontend dependencies
cd static/hello-world
npm install
cd ../..
```

#### 3. Forge Authentication
```bash
# Login to Forge
forge login

# Verify authentication
forge whoami
```

#### 4. Configure Development Environment
```bash
# Create development app
forge register

# Set up environment variables
cp .env.example .env
# Edit .env with your configuration
```

## Project Structure

```
quality-compass/
├── src/
│   ├── index.js                 # Forge resolvers (backend)
│   ├── validators/              # Input validation
│   │   └── inputValidator.js
│   ├── middleware/              # Middleware functions
│   │   └── rateLimiter.js
│   ├── utils/                   # Utility functions
│   │   └── errorHandler.js
│   └── services/                # Service layer
│       ├── auditService.js
│       └── gdprService.js
│
├── src-frontend/                # Frontend React app
│   ├── src/
│   │   ├── components/          # React components
│   │   │   ├── admin/          # Admin dashboard components
│   │   │   ├── shared/         # Shared components
│   │   │   └── evaluation/    # Evaluation components
│   │   ├── services/           # Frontend services
│   │   │   ├── analyticsService.ts
│   │   │   ├── exportService.ts
│   │   │   └── cacheService.ts
│   │   ├── contexts/           # React contexts
│   │   ├── hooks/              # Custom React hooks
│   │   ├── types/              # TypeScript definitions
│   │   ├── pages/              # Page components
│   │   └── App.tsx             # Main app component
│   │
│   ├── public/                 # Static assets
│   ├── vite.config.ts          # Vite configuration
│   ├── tsconfig.json           # TypeScript configuration
│   └── package.json            # Frontend dependencies
│
├── manifest.yml                # Forge app manifest
├── package.json               # Root dependencies
├── .gitignore
└── README.md
```

## Development Workflow

### 1. Local Development with Tunnel

```bash
# Start the tunnel for hot-reloading
forge tunnel

# In another terminal, start the frontend dev server (if applicable)
npm run dev
```

### 2. Making Changes

#### Backend Changes (Resolvers)
```javascript
// src/index.js
resolver.define('newFunction', async (req) => {
  // Validate input
  const { param } = req.payload;
  validateInput(param);
  
  // Business logic
  const result = await processData(param);
  
  // Return response
  return result;
});
```

#### Frontend Changes (React)
```typescript
// src/components/NewComponent.tsx
import React from 'react';
import { invoke } from '@forge/bridge';

export const NewComponent: React.FC = () => {
  const fetchData = async () => {
    const result = await invoke('newFunction', { param: 'value' });
    // Handle result
  };
  
  return <div>Component Content</div>;
};
```

### 3. Building for Production

```bash
# Build frontend
npm run build

# Deploy to staging
cd ../..
forge deploy --environment staging

# Deploy to production
forge deploy --environment production
```

For production deployment details, see [Installation Guide](./Installation.md#installation-steps).

## Code Standards

### JavaScript/TypeScript Style Guide

```typescript
// Use TypeScript for type safety
interface EvaluationData {
  ticketId: string;
  score: number;
  criteria: Record<string, number>;
}

// Use async/await over promises
const fetchData = async (): Promise<EvaluationData> => {
  try {
    const data = await invoke('getData');
    return data;
  } catch (error) {
    console.error('Error fetching data:', error);
    throw error;
  }
};

// Use functional components with hooks
const MyComponent: React.FC<Props> = ({ prop1, prop2 }) => {
  const [state, setState] = useState<string>('');
  
  useEffect(() => {
    // Effect logic
  }, [dependency]);
  
  return <div>{state}</div>;
};
```

### Naming Conventions

```typescript
// Files: kebab-case
evaluation-form.tsx
analytics-service.ts

// Components: PascalCase
EvaluationForm
AnalyticsDashboard

// Functions/Variables: camelCase
const calculateAverage = (scores: number[]): number => {};
let evaluationCount = 0;

// Constants: UPPER_SNAKE_CASE
const MAX_RETRIES = 3;
const API_TIMEOUT = 5000;

// Types/Interfaces: PascalCase
interface UserData {}
type ScoreRange = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10;
```

## Testing

### Unit Testing

```typescript
// src/services/__tests__/analyticsService.test.ts
import { calculateTrend } from '../analyticsService';

describe('Analytics Service', () => {
  describe('calculateTrend', () => {
    it('should identify improving trend', () => {
      const recent = [8, 9, 9];
      const historical = [6, 7, 6];
      const result = calculateTrend(recent, historical);
      expect(result.direction).toBe('improving');
      expect(result.percentage).toBeGreaterThan(0);
    });
  });
});
```

### Integration Testing

```javascript
// src/__tests__/resolver.test.js
const { resolver } = require('../index');

describe('Resolver Integration', () => {
  it('should fetch issue data', async () => {
    const req = {
      payload: { issueIdOrKey: 'TEST-123' }
    };
    
    const result = await resolver.getIssue(req);
    expect(result).toHaveProperty('key', 'TEST-123');
  });
});
```

### Running Tests

```bash
# Run all tests
npm test

# Run with coverage
npm run test:coverage

# Run specific test file
npm test -- analyticsService.test.ts

# Run in watch mode
npm run test:watch
```

## Debugging

### Using Forge Logs

```bash
# View real-time logs
forge logs

# View logs for specific environment
forge logs --environment production

# Filter logs
forge logs --filter error
```

### Browser DevTools

```javascript
// Add debug logging
console.log('[Quality Compass] Data loaded:', data);

// Use breakpoints
debugger; // Execution will pause here

// Performance profiling
console.time('DataFetch');
await fetchData();
console.timeEnd('DataFetch');
```

### Common Debugging Scenarios

#### Issue: Resolver not found
```javascript
// Check manifest.yml
resolver:
  function: resolver
  
// Verify function is defined
resolver.define('functionName', async (req) => {});
```

For more debugging scenarios, see [Technical Troubleshooting](./Troubleshooting-Technical.md).

#### Issue: Storage errors
```javascript
// Add detailed logging
try {
  const data = await storage.get('key');
  console.log('Storage data:', data);
} catch (error) {
  console.error('Storage error:', error);
  // Check storage quota
  // Verify key format
}
```

## Performance Optimization

### Frontend Optimization

```typescript
// Use React.memo for expensive components
const ExpensiveComponent = React.memo(({ data }) => {
  return <ComplexVisualization data={data} />;
});

// Implement lazy loading
const AdvancedAnalytics = React.lazy(() => 
  import('./components/AdvancedAnalytics')
);

// Use useMemo for expensive calculations
const processedData = useMemo(() => 
  processComplexData(rawData), 
  [rawData]
);

// Debounce user input
const debouncedSearch = useMemo(
  () => debounce(handleSearch, 300),
  []
);
```

### Backend Optimization

```javascript
// Batch API calls
const batchFetch = async (ids) => {
  const chunks = chunk(ids, 50);
  const results = await Promise.all(
    chunks.map(chunk => fetchBatch(chunk))
  );
  return results.flat();
};

// Implement caching
const cache = new Map();
const getCachedData = async (key) => {
  if (cache.has(key)) {
    return cache.get(key);
  }
  const data = await fetchData(key);
  cache.set(key, data);
  return data;
};
```

## Security Best Practices

### Input Validation

```javascript
// Always validate and sanitize input
const validateInput = (input) => {
  // Check type
  if (typeof input !== 'string') {
    throw new Error('Invalid input type');
  }
  
  // Check length
  if (input.length > MAX_LENGTH) {
    throw new Error('Input too long');
  }
  
  // Sanitize
  return input.replace(/[<>]/g, '');
};
```

### Secure Storage

```javascript
// Never store sensitive data in frontend
// Use Forge storage for sensitive information
await storage.setSecret('api-key', secretValue);

// Implement access controls
const canAccess = await checkPermissions(user, resource);
if (!canAccess) {
  throw new Error('Access denied');
}
```

For complete security guidelines, see [Security Documentation](./Security.md).

## Deployment

### Deployment Checklist

```bash
# Pre-deployment
□ Run all tests
□ Update version in package.json
□ Update CHANGELOG.md
□ Review security vulnerabilities
□ Build production bundle
□ Test in staging environment

# Deployment
forge deploy --environment production

# Post-deployment
□ Verify deployment
□ Run smoke tests
□ Monitor logs
□ Update documentation
```

### Environment Management

```yaml
# manifest.yml - Environment-specific configuration
app:
  id: ${APP_ID}
  
environments:
  development:
    variables:
      LOG_LEVEL: debug
      
  staging:
    variables:
      LOG_LEVEL: info
      
  production:
    variables:
      LOG_LEVEL: error
```

## Monitoring

### Application Metrics

```javascript
// Track performance metrics
const trackMetric = (name, value) => {
  console.log(`[METRIC] ${name}: ${value}`);
  // Send to monitoring service
};

// Usage
const startTime = Date.now();
await performOperation();
trackMetric('operation.duration', Date.now() - startTime);
```

### Error Tracking

```javascript
// Centralized error handling
const handleError = (error, context) => {
  console.error('[ERROR]', {
    message: error.message,
    stack: error.stack,
    context
  });
  
  // Send to error tracking service
  // Notify administrators if critical
};
```

## Contributing

### Pull Request Process

1. **Create feature branch**
   ```bash
   git checkout -b feature/your-feature
   ```

2. **Make changes and commit**
   ```bash
   git add .
   git commit -m "feat: add new feature"
   ```

3. **Run tests and linting**
   ```bash
   npm run test
   npm run lint
   ```

4. **Push and create PR**
   ```bash
   git push origin feature/your-feature
   ```

### Commit Message Convention

```
type(scope): subject

body

footer
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Code style
- `refactor`: Refactoring
- `test`: Tests
- `chore`: Maintenance

## Resources

### Documentation
- [Forge Documentation](https://developer.atlassian.com/platform/forge/)
- [Jira Cloud REST API](https://developer.atlassian.com/cloud/jira/platform/rest/v3/)
- [React Documentation](https://react.dev/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

### Tools
- [Forge CLI Reference](https://developer.atlassian.com/platform/forge/cli-reference/)
- [VS Code Forge Extension](https://marketplace.visualstudio.com/items?itemName=atlassian.atlascode)
- [React DevTools](https://react.dev/learn/react-developer-tools)

---

*Last Updated: January 2025*

