# Business Logic Model — Foundation Unit

## Auth Flows

### Registration Flow
```
1. User enters email + password
2. Frontend validates: email format, password policy (BR-01)
3. Frontend calls POST /auth/register { email, password }
4. Auth Service → Cognito: signUp(email, password)
5. Cognito sends verification code to email
6. User enters code
7. Frontend calls POST /auth/confirm { email, code }
8. Auth Service → Cognito: confirmSignUp(email, code)
9. Auth Service → DynamoDB: create UserProfile (default values)
10. Return success → redirect to Profile Setup wizard
```

### Login Flow
```
1. User enters email + password
2. Frontend validates: non-empty fields
3. Frontend calls POST /auth/login { email, password }
4. Auth Service → Cognito: initiateAuth(email, password)
5. Cognito returns: { accessToken, refreshToken, idToken }
6. Frontend stores tokens (accessToken in memory, refreshToken in secure storage)
7. Frontend decodes idToken for user info (userId, email)
8. Redirect to Dashboard (or Profile Setup if profileCompleted=false)
```

### Token Refresh Flow
```
1. Frontend detects accessToken expiring (< 5 min remaining)
2. Frontend calls POST /auth/refresh { refreshToken }
3. Auth Service → Cognito: initiateAuth(REFRESH_TOKEN, refreshToken)
4. Cognito returns: { accessToken, idToken } (new tokens)
5. Frontend updates stored tokens
6. If refresh fails (expired/revoked): redirect to login
```

### Password Reset Flow
```
1. User clicks "Forgot Password"
2. User enters email
3. Frontend calls POST /auth/forgot-password { email }
4. Auth Service → Cognito: forgotPassword(email)
5. Cognito sends 6-digit code to email
6. User enters code + new password
7. Frontend validates: new password meets policy (BR-01)
8. Frontend calls POST /auth/confirm-password { email, code, newPassword }
9. Auth Service → Cognito: confirmForgotPassword(email, code, newPassword)
10. Return success → redirect to login
```

### Logout Flow
```
1. User clicks "Logout"
2. Frontend calls POST /auth/logout { refreshToken }
3. Auth Service → Cognito: globalSignOut(accessToken)
4. Frontend clears all stored tokens
5. Frontend clears PouchDB user data (optional, based on "remember me")
6. Redirect to login page
```

---

## Profile Management Logic

### Get Profile
```
1. Frontend calls GET /auth/profile (with accessToken)
2. API Gateway validates JWT (Cognito authorizer)
3. Auth Service extracts userId from event.requestContext
4. Auth Service → DynamoDB: get(PK=USER#{userId}, SK=PROFILE#)
5. Return UserProfile (or 404 if not found)
```

### Update Profile
```
1. Frontend validates fields per BR-04 constraints
2. Frontend calls PUT /auth/profile { updates }
3. Auth Service validates all fields server-side (BR-04)
4. Auth Service → DynamoDB: update(PK=USER#{userId}, SK=PROFILE#, updates)
5. Set updatedAt = now()
6. If dailyCalorieTarget is set and profileCompleted=false: set profileCompleted=true
7. Return updated UserProfile
```

---

## Offline Sync Algorithm

### Initialization (App Start)
```
1. Check navigator.onLine status
2. Initialize PouchDB local database
3. Load cached data from PouchDB
4. If online: trigger sync
5. Register online/offline event listeners
```

### Mutation Queue (Offline Write)
```
1. User performs action (e.g., log meal, update profile)
2. Save to PouchDB with: { entityType, entityId, operation, payload, timestamp, synced: false }
3. Update local UI immediately (optimistic)
4. Increment pendingCount in SyncState
```

### Sync Process (On Reconnect)
```
1. Online event detected (or manual trigger)
2. Set syncStatus = "syncing"
3. Query PouchDB for all records where synced=false, ordered by timestamp ASC
4. For each pending mutation:
   a. Send to appropriate API endpoint (POST/PUT/DELETE)
   b. Include lastKnownVersion timestamp in request
   c. If success (200/201):
      - Mark record as synced=true
      - Update local entity with server response
      - Decrement pendingCount
   d. If conflict (409):
      - Server-wins: overwrite local with server version
      - Mark record as synced=true (discard local change)
      - Show toast: "Change overwritten by server"
      - Decrement pendingCount
   e. If error (5xx/network):
      - Retry up to 3 times (exponential backoff: 1s, 2s, 4s)
      - If all retries fail: set syncStatus="error", stop sync
      - Leave record as synced=false for next attempt
5. After all mutations processed:
   - Set syncStatus = "idle"
   - Set lastSyncTimestamp = now()
   - Fetch latest data from server (pull)
   - Update PouchDB with fresh server data
```

### Pull Sync (Fetch Latest)
```
1. After push sync completes
2. Call GET endpoints for user's data (profile, meals, ingredients)
3. Compare server timestamps with local PouchDB timestamps
4. Update local PouchDB with any newer server records
5. This handles changes made on other devices
```

---

## Shared UI Logic

### Theme Management
```
1. Check localStorage for saved theme preference
2. If none: use system preference (prefers-color-scheme)
3. Apply theme class to document root
4. Provide toggle function via React Context
```

### Navigation Logic
```
1. If not authenticated: show login/register routes only
2. If authenticated but profileCompleted=false: redirect to profile setup
3. If authenticated and profileCompleted=true: show full app navigation
4. Navigation items: Dashboard, Log Meal, Ingredients, Recipes, Profile
```

### Error Boundary Logic
```
1. Catch unhandled errors in React component tree
2. Log error details (component stack, error message)
3. Show user-friendly error message with "Try Again" button
4. Provide "Report Issue" link (future)
```
