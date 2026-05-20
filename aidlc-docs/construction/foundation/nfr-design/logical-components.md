# Logical Components — Foundation Unit

## Backend Logical Components

### 1. Lambda Handler Wrapper
**Purpose**: Consistent request/response handling across all Lambda functions  
**Responsibilities**:
- Parse event body (JSON)
- Extract userId from Cognito authorizer context
- Call service function
- Format success response (200/201 + body)
- Catch errors, log (ERROR level), return appropriate HTTP status + message
- Add CORS headers

```typescript
// Pattern: src/backend/shared/handler.ts
type HandlerFn = (event: APIGatewayEvent, userId: string) => Promise<{ statusCode: number; body: any }>;

function createHandler(fn: HandlerFn) {
  return async (event: APIGatewayEvent) => {
    try {
      const userId = event.requestContext.authorizer?.claims?.sub;
      const result = await fn(event, userId);
      return { statusCode: result.statusCode, headers: corsHeaders, body: JSON.stringify(result.body) };
    } catch (error) {
      console.error(JSON.stringify({ level: 'ERROR', error: error.message, requestId: event.requestContext.requestId }));
      return { statusCode: 500, headers: corsHeaders, body: JSON.stringify({ message: 'Internal server error' }) };
    }
  };
}
```

### 2. DynamoDB Client Wrapper
**Purpose**: Simplified DynamoDB operations with consistent error handling  
**Responsibilities**:
- Initialize DynamoDB Document Client (singleton)
- Provide typed get/put/update/delete/query operations
- Handle DynamoDB-specific errors (ConditionalCheckFailed, Throttled)
- Add updatedAt timestamp on writes

### 3. Validation Utility
**Purpose**: Server-side input validation  
**Responsibilities**:
- Validate request body fields against business rules (BR-04)
- Return first validation error found
- Reusable validators: email, string length, number range, enum, array

### 4. Logger Utility
**Purpose**: Structured JSON logging  
**Responsibilities**:
- Log at ERROR level only (production)
- Include: service name, method, requestId, timestamp
- Mask PII (emails, names)
- No-op for non-ERROR levels in production

---

## Frontend Logical Components

### 1. API Client (axios instance)
**Purpose**: Configured HTTP client for all backend calls  
**Responsibilities**:
- Base URL configuration (API Gateway endpoint)
- Auto-attach Authorization header (access token from AuthProvider)
- Response interceptor: on 401 → attempt token refresh → retry original request
- Request timeout: 10 seconds
- Retry interceptor: 3 retries with exponential backoff for 5xx/network errors

```typescript
// Pattern: src/frontend/shared/api-client.ts
const apiClient = axios.create({ baseURL: API_URL, timeout: 10000 });

apiClient.interceptors.request.use((config) => {
  const token = getAccessToken(); // from AuthProvider
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) { /* refresh token logic */ }
    if (error.response?.status >= 500) { /* retry logic */ }
    return Promise.reject(error);
  }
);
```

### 2. AuthProvider (React Context)
**Purpose**: Global authentication state and operations  
**State**:
- user: User | null
- isAuthenticated: boolean
- isLoading: boolean (initial auth check)

**Actions**:
- login(email, password) → store tokens, set user
- register(email, password) → initiate registration
- logout() → clear tokens, clear state
- refreshSession() → refresh access token silently

**Lifecycle**:
- On mount: check for existing refresh token → attempt silent refresh
- On token expiry (5 min before): auto-refresh in background
- On 401 from API: attempt refresh → if fail: logout

### 3. SyncProvider (React Context)
**Purpose**: Offline sync state and operations  
**State**:
- isOnline: boolean (navigator.onLine)
- syncStatus: "idle" | "syncing" | "error" | "offline"
- pendingChanges: number

**Actions**:
- queueMutation(entityType, entityId, operation, payload) → save to PouchDB
- triggerSync() → push pending, pull latest
- clearCache() → wipe PouchDB (on logout)

**Lifecycle**:
- On mount: register online/offline event listeners
- On online event: auto-trigger sync
- On offline event: set syncStatus="offline"

### 4. ThemeProvider (React Context)
**Purpose**: Dark/light theme management  
**State**:
- theme: "light" | "dark"

**Actions**:
- toggleTheme() → switch and persist to localStorage

**Lifecycle**:
- On mount: read localStorage → fallback to system preference (prefers-color-scheme)

### 5. useApi Hook (Custom Hook Pattern)
**Purpose**: Reusable data fetching pattern for all API calls  
**Pattern**:
```typescript
function useApi<T>(fetcher: () => Promise<T>) {
  const [data, setData] = useState<T | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  // ... fetch logic with loading/error states
  return { data, isLoading, error, refetch };
}
```

**Offline-aware**: If offline, read from PouchDB cache instead of API call.

### 6. PouchDB Service
**Purpose**: Local database management  
**Responsibilities**:
- Initialize PouchDB database per user (namespace by userId)
- CRUD operations on local documents
- Query by entityType and date range
- Manage sync queue (unsynced mutations)
- Cache cleanup (remove data older than 30 days)
- Destroy database on logout

---

## Component Interaction Diagram

```
Frontend:
+------------------+     +------------------+     +------------------+
|   UI Components  | --> |   Custom Hooks   | --> |   API Client     |
|   (React)        |     |   (useApi, etc.) |     |   (axios)        |
+------------------+     +------------------+     +------------------+
        |                        |                        |
        v                        v                        v
+------------------+     +------------------+     +------------------+
|   Context        |     |   PouchDB        |     |   API Gateway    |
|   Providers      |     |   Service        |     |   (REST)         |
| (Auth,Sync,Theme)|     |   (local cache)  |     |                  |
+------------------+     +------------------+     +------------------+

Backend:
+------------------+     +------------------+     +------------------+
|   API Gateway    | --> | Handler Wrapper  | --> | Service Logic    |
|   + Cognito Auth |     | (parse, validate)|     | (business rules) |
+------------------+     +------------------+     +------------------+
                                                         |
                                                         v
                                                  +------------------+
                                                  | DynamoDB Client  |
                                                  | (typed wrapper)  |
                                                  +------------------+
```
