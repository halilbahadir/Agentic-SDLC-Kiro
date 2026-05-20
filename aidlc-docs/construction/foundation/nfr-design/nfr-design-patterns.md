# NFR Design Patterns — Foundation Unit

## 1. Resilience Patterns

### Retry Pattern (Frontend → Backend)
**Where**: axios HTTP client (frontend)  
**Strategy**: Exponential backoff for 5xx errors and network failures

| Attempt | Delay | Condition |
|---------|-------|-----------|
| 1st retry | 1 second | 5xx or network error |
| 2nd retry | 2 seconds | 5xx or network error |
| 3rd retry | 4 seconds | 5xx or network error |
| Give up | — | Show error to user |

**Not retried**: 4xx errors (client errors are not transient)

### Timeout Pattern
| Operation | Timeout | Action on Timeout |
|-----------|---------|-------------------|
| API calls (frontend) | 10 seconds | Show error, offer retry |
| Lambda execution | 30 seconds | API Gateway returns 504 |
| Cognito operations | 10 seconds | Lambda timeout handling |
| DynamoDB operations | 5 seconds | Lambda returns 500 |

### Graceful Degradation Pattern
| Scenario | Degraded Behavior |
|----------|-------------------|
| Backend unavailable | Show cached data from PouchDB, queue mutations |
| Cognito unavailable | Show "Service temporarily unavailable", keep existing session |
| DynamoDB throttled | Lambda retries with SDK built-in backoff |
| S3 unavailable | Show placeholder for images, disable upload |

---

## 2. Performance Patterns

### Code Splitting (Frontend)
**Pattern**: React.lazy() + Suspense for route-level splitting

```
Bundle structure:
- main.js          (shared: React, Router, Auth, Shared UI)
- meal-logging.js  (lazy: Meal Logging module)
- dashboard.js     (lazy: Nutrition Dashboard module)
- ingredients.js   (lazy: Ingredient Management module)
- recipes.js       (lazy: Recipe Advisor module)
```

**Loading**: Show LoadingSpinner component during chunk load.

### Caching Strategy
| Layer | Cache | TTL | Invalidation |
|-------|-------|-----|--------------|
| CloudFront (static assets) | Edge cache | 1 year (versioned filenames) | Deploy new version |
| CloudFront (API) | No cache | — | Pass-through to API Gateway |
| PouchDB (local data) | IndexedDB | 30 days | Sync on reconnect |
| Browser (auth tokens) | Memory | 1 hour (access), 30 days (refresh) | Logout or expiry |

### Lazy Loading (Images)
**Pattern**: Intersection Observer for below-fold images  
**Placeholder**: Low-res blur or skeleton while loading

---

## 3. Security Patterns

### Token Management (Frontend)
```
┌─────────────────────────────────────────────┐
│ AuthProvider (React Context)                 │
│                                             │
│  accessToken: in-memory (React state)       │
│  refreshToken: httpOnly cookie              │
│  idToken: in-memory (user info)             │
│                                             │
│  Auto-refresh: 5 min before expiry          │
│  On 401: attempt refresh → if fail: logout  │
└─────────────────────────────────────────────┘
```

### Input Sanitization
| Layer | What | How |
|-------|------|-----|
| Frontend | Form inputs | Trim whitespace, validate format before submit |
| Backend | All request bodies | Validate types, lengths, allowed values per BR-04 |
| Backend | Query params | Validate and sanitize (no SQL/NoSQL injection risk with DynamoDB, but validate types) |

### CORS Configuration
```
Allowed Origins: [app domain only]
Allowed Methods: GET, POST, PUT, DELETE, OPTIONS
Allowed Headers: Content-Type, Authorization
Max Age: 86400 (24 hours)
Credentials: true (for httpOnly cookies)
```

### API Gateway Authorization
```
All /auth/register, /auth/login, /auth/confirm, /auth/forgot-password, /auth/confirm-password
  → No authorizer (public)

All other routes
  → Cognito User Pool Authorizer (JWT validation)
  → userId extracted from token claims
```

---

## 4. Observability Patterns

### Structured Logging (Backend)
**Level**: ERROR only in production  
**Format**: JSON structured logs

```json
{
  "level": "ERROR",
  "service": "auth-service",
  "method": "loginUser",
  "userId": "anonymous",
  "error": "InvalidCredentials",
  "message": "Login failed for email: u***@example.com",
  "requestId": "abc-123",
  "timestamp": "2026-05-17T00:00:00Z"
}
```

**Rules**:
- Never log passwords, tokens, or full email addresses
- Mask PII in error logs (first char + *** + domain)
- Include requestId for correlation
- Include Lambda request ID for CloudWatch correlation

### CloudWatch Alarms
| Alarm | Metric | Threshold | Action |
|-------|--------|-----------|--------|
| Auth Errors | Lambda errors (auth function) | > 5 in 5 min | SNS notification |
| API 5xx Rate | API Gateway 5xx count | > 10 in 5 min | SNS notification |
| High Latency | API Gateway p95 latency | > 5 seconds | SNS notification |
| DynamoDB Throttle | ThrottledRequests | > 0 in 5 min | SNS notification |

### CloudWatch Dashboard
Single dashboard with panels:
- API Gateway: request count, 4xx/5xx rates, latency (p50/p95)
- Lambda: invocations, errors, duration per function
- DynamoDB: read/write capacity consumed, throttles
- Cognito: sign-up/sign-in counts

---

## 5. Offline Sync Pattern (Detailed)

### Write Path (Offline)
```
User Action → SyncProvider.queueMutation()
  → PouchDB.put({ entityType, entityId, operation, payload, timestamp, synced: false })
  → Update UI optimistically
  → Increment pendingCount
```

### Sync Path (On Reconnect)
```
navigator.onLine event → SyncProvider.triggerSync()
  → Set syncStatus = "syncing"
  → Query PouchDB: synced=false, ORDER BY timestamp ASC
  → For each mutation:
      → POST/PUT/DELETE to API
      → If 200: mark synced=true, update local entity
      → If 409 (conflict): overwrite local with server response (server-wins)
      → If 5xx: retry 3x → if fail: stop, set syncStatus="error"
  → After all pushed: pull latest (GET endpoints)
  → Update PouchDB with server data
  → Set syncStatus = "idle"
```

### Read Path (Offline)
```
Component requests data → Custom hook
  → If online: fetch from API, update PouchDB cache, return data
  → If offline: read from PouchDB, return cached data
  → Show stale indicator if cache > 24h old
```
