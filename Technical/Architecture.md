# Technical Architecture

## Overview

The Quality Compass is built on the Atlassian Forge platform, providing a comprehensive quality assurance evaluation system integrated directly into Jira. This document describes the technical architecture, system components, and design decisions.

**Quick Links:**
- [Installation Guide](./Installation.md) - Get started with deployment
- [API Reference](./API-Reference.md) - Detailed API documentation
- [Security Documentation](./Security.md) - Security architecture and compliance
- [Development Guide](./Development.md) - Development setup and practices

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Jira Cloud                           │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              Forge Platform Runtime                  │  │
│  │  ┌─────────────────────────────────────────────┐   │  │
│  │  │          Quality Compass                        │   │  │
│  │  │  ┌──────────────┐  ┌──────────────────┐   │   │  │
│  │  │  │   Frontend   │  │    Backend       │   │   │  │
│  │  │  │   (React)    │  │   (Resolvers)    │   │   │  │
│  │  │  └──────────────┘  └──────────────────┘   │   │  │
│  │  │           │               │                │   │  │
│  │  │           └───────┬───────┘                │   │  │
│  │  │                   │                        │   │  │
│  │  │         ┌─────────▼─────────┐             │   │  │
│  │  │         │   Forge Storage   │             │   │  │
│  │  │         └───────────────────┘             │   │  │
│  │  └─────────────────────────────────────────────┘   │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## Technology Stack

### Frontend
- **Framework**: React 18.x with TypeScript
- **Build Tool**: Vite
- **UI Components**: 
  - shadcn/ui (Radix UI primitives)
  - Tailwind CSS for styling
- **State Management**: React Context API
- **Data Visualization**: Recharts
- **Date Handling**: date-fns
- **Excel Export**: exceljs
- **Workers**: Web Worker for CPU-heavy advanced analytics
- **Persistent Caching**: IndexedDB (SWR pattern)

### Backend
- **Runtime**: Forge Platform (Node.js-based)
- **Language**: JavaScript (ES6+)
- **API Integration**: Forge API for Jira interaction
- **Storage**: Forge Storage API
- **Authentication**: Forge authentication (api.asUser, api.asApp)
- **Request Management**: In-flight request deduplication + client concurrency limiter

### Security
- **Input Validation**: Custom validation middleware
- **Rate Limiting**: In-memory rate limiter
- **CSP**: Content Security Policy configured
- **Audit Logging**: Custom audit service
- **GDPR Compliance**: Built-in data management features

## Core Components

### 1. Frontend Components

#### Main Views
- **AdminView**: Central dashboard for administrators
- **AnalyticsDashboard**: Comprehensive analytics and reporting
- **AgentPerformanceDashboard**: Agent-specific performance metrics
- **EvaluatorAnalyticsDashboard**: Evaluator performance analysis
- **AdvancedAnalyticsDashboard**: Advanced analytics with predictions

#### Key Services
```typescript
// Service Layer Architecture
├── analyticsService.ts      // Core analytics calculations
├── exportService.ts         // Export functionality (CSV, Excel)
├── advancedAnalyticsService.ts // Advanced analytics and ML features
├── cacheService.ts          // Client-side caching
└── utils/
    └── storageUtils.ts      // Storage utilities
```

### 2. Backend Resolvers

Located in `src/index.js`, the resolvers handle:

```javascript
// Core Resolver Functions
resolver.define('makeApiRequest', async (req) => {});
resolver.define('getIssue', async (req) => {});
resolver.define('searchIssues', async (req) => {});
resolver.define('getStorageValue', async (req) => {});
resolver.define('setStorageValue', async (req) => {});
resolver.define('deleteStorageValue', async (req) => {});
resolver.define('batchGetUsers', async (req) => {});
resolver.define('getUser', async (req) => {});
resolver.define('setEvaluatorName', async (req) => {});
```

### 3. Data Models

#### Evaluation Schema
```typescript
interface TicketEvaluation {
  ticketId: string;
  ticketKey: string;
  evaluatorId: string;
  evaluatorDisplayName?: string;
  averageScore: number;
  criteria: Record<string, number>;
  feedback?: string;
  timestamp: string;
  projectKey?: string;
  priority?: string;
  issueType?: string;
  status?: string;
  assigneeId?: string;
  assigneeDisplayName?: string;
}
```

#### Storage Structure (actual keys)
```javascript
// Forge Storage Keys (used by app)
'ticket-evaluations'                 // Global list of evaluations
'project-evaluations-<PROJECT_KEY>'  // Project-scoped evaluations
'evaluation-criteria'                // Criteria configuration
'evaluator-name-mapping'             // Manual evaluator name overrides
'user-displayname-cache'             // Auto-resolved display names cache
'selected-project'                   // Last selected project key per user
'coverage-targets'                   // Admin-configured coverage targets
```

## Data Flow

### 1. Evaluation Creation Flow
```
User Action → Frontend Form → Validation → Forge Resolver 
→ Storage API → Cache Invalidation → UI Update
```

### 2. Analytics Generation Flow
```
Dashboard Request → Cache Check → Data Fetch from Storage 
→ Analytics Calculation → Data Transformation → UI Render
```

### 3. Export Flow
```
Export Request → Data Collection → Format Transformation 
→ File Generation → Download Trigger
```

## Integration Points

### Jira REST API Integration
- **Issue Retrieval**: `/rest/api/3/issue/{issueIdOrKey}`
- **Search**: `/rest/api/3/search` (supports `startAt` pagination and `nextPageToken` where available)
- **User Management**: `/rest/api/3/user`
- **Project Access**: `/rest/api/3/project`

### Forge Storage API
- **Key-Value Storage**: For evaluations and configuration
- **Batch Operations**: For performance optimization
- **Query Operations**: For filtered data retrieval

For detailed API documentation, see [API Reference](./API-Reference.md#storage-api-functions).

## Performance Considerations

### Caching Strategy
- **Client-side**: In-memory cache with TTL + IndexedDB persistent cache (SWR)
- **Cache Keys**: Generated based on request parameters
- **TTL Values (current)**:
  - SHORT: 1 minute
  - MEDIUM: 5 minutes
  - LONG: 30 minutes
  - VERY_LONG: 2 hours (advanced analytics)
- **Persistent Cache**: Advanced analytics results persisted and refreshed in background

### Optimization Techniques
1. **Batch API Calls**: Reduce network overhead
2. **Lazy Loading**: Load data on demand
3. **Memoization**: Cache computed values
4. **Virtual Scrolling**: For large data sets
5. **Debouncing**: For search and filter inputs
6. **In-flight Request Deduplication**: Identical calls coalesce
7. **Global Concurrency Limiter**: Max 8 Forge invocations at once
8. **Adjacent Tab Prefetch Constraints**: Heavy tabs not prefetched in background
9. **Worker Offload**: CPU-heavy analytics runs in Web Worker

## Security Architecture

### Input Validation
```javascript
// Validation Pipeline
Input → Sanitization → Type Checking → Range Validation 
→ Pattern Matching → Business Logic Validation
```

For complete security implementation, see [Security Documentation](./Security.md#input-validation).

### Authentication Flow
```
User Request → Forge Context → Permission Check 
→ Scoped API Access → Response
```

### Data Protection
- **At Rest**: Encrypted via Forge Storage
- **In Transit**: HTTPS enforced
- **Access Control**: Role-based permissions
- **Audit Trail**: All modifications logged

## Scalability Design

### Horizontal Scaling
- Stateless resolver design
- Distributed caching capability
- Async processing for heavy operations

### Vertical Scaling
- Efficient memory usage
- Optimized database queries
- Minimal computational overhead

## Monitoring & Observability

### Logging Strategy
- **Error Logging**: Comprehensive error capture
- **Audit Logging**: Security and compliance events
- **Performance Logging**: Response time tracking
- **Debug Logging**: Development environment only

### Metrics Collection
- API response times
- Storage operation performance
- User action tracking
- Error rates and types

## Deployment Architecture

### Forge Deployment
```yaml
# manifest.yml structure
app:
  id: ari:cloud:ecosystem::app/{app-id}
  
modules:
  jira:projectPage:
    - key: qa-app-project-page
      resource: main
      resolver:
        function: resolver
        
resources:
  - key: main
    path: build
    
permissions:
  scopes:
    - read:jira-work
    - write:jira-work
    - storage:app
```

### Build Pipeline
1. **Development**: Local development with tunnel
2. **Build**: Vite production build (workers via `new URL('./worker.ts', import.meta.url)` and `vite.config.ts` `worker.format = 'es'`)
3. **Deploy**: Forge deploy command
4. **Verification**: Automated smoke tests

## Disaster Recovery

### Backup Strategy
- **Data Backup**: Regular export of evaluations
- **Configuration Backup**: Settings and criteria
- **Recovery Time Objective**: < 4 hours
- **Recovery Point Objective**: < 24 hours

### Failover Procedures
1. Forge platform handles infrastructure failover
2. Application-level retry logic
3. Graceful degradation for non-critical features

## Future Architecture Considerations

### Planned Enhancements
- WebSocket support for real-time updates
- Machine learning model integration
- External API webhooks
- Multi-region deployment
- Advanced caching with Redis

### Scalability Roadmap
- Microservices architecture consideration
- Event-driven processing
- GraphQL API implementation
- Containerization for local development

---

## Appendix

### File Structure
```
quality-compass/
├── src/
│   ├── index.js              # Forge resolvers
│   ├── components/           # React components
│   ├── services/            # Business logic
│   ├── contexts/            # React contexts
│   ├── hooks/               # Custom hooks
│   ├── types/               # TypeScript definitions
│   └── validators/          # Input validation
├── build/                    # Frontend build output
├── manifest.yml             # Forge configuration
└── package.json            # Dependencies
```

### Key Design Decisions

1. **Forge Platform**: Chosen for native Jira integration and security
2. **React + TypeScript**: Type safety and component reusability
3. **Vite**: Fast build times and modern tooling
4. **ExcelJS**: Robust Excel generation without vulnerabilities
5. **Context API**: Simple state management without external dependencies

## See Also

- [Installation Guide](./Installation.md) - Setup and deployment instructions
- [API Reference](./API-Reference.md) - Complete API documentation
- [Development Guide](./Development.md) - Development workflow and standards
- [Security Documentation](./Security.md) - Security measures and compliance
- [Technical Troubleshooting](./Troubleshooting-Technical.md) - Common issues and solutions

---

*Last Updated: January 2025*

