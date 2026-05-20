# Business Rules — Foundation Unit

## BR-01: Registration Validation

| Rule | Constraint | Error Code |
|------|-----------|------------|
| Email format | Must be valid email (RFC 5322) | INVALID_EMAIL |
| Email uniqueness | No existing account with same email | EMAIL_ALREADY_EXISTS |
| Password length | Minimum 8 characters | PASSWORD_TOO_SHORT |
| Password uppercase | At least 1 uppercase letter | PASSWORD_MISSING_UPPERCASE |
| Password lowercase | At least 1 lowercase letter | PASSWORD_MISSING_LOWERCASE |
| Password number | At least 1 digit | PASSWORD_MISSING_NUMBER |
| Password special | At least 1 special character (!@#$%^&*) | PASSWORD_MISSING_SPECIAL |
| Email confirmation | Must verify email before full access | EMAIL_NOT_VERIFIED |

---

## BR-02: Login Rules

| Rule | Constraint | Error Code |
|------|-----------|------------|
| Credentials match | Email + password must match Cognito record | INVALID_CREDENTIALS |
| Email verified | Account must have verified email | EMAIL_NOT_VERIFIED |
| Account active | Account must not be disabled | ACCOUNT_DISABLED |
| Rate limiting | Max 5 failed attempts per 15 minutes | TOO_MANY_ATTEMPTS |

---

## BR-03: Session Management

| Rule | Value | Description |
|------|-------|-------------|
| Access token lifetime | 1 hour | Short-lived, auto-refreshed |
| Refresh token lifetime | 30 days | Expires after 30 days of inactivity |
| Token refresh | Automatic | Frontend refreshes before access token expires |
| Logout | Invalidate refresh token | Revoke on server side |
| Multi-device | Allowed | Same account can be logged in on multiple devices |

---

## BR-04: Profile Constraints

| Field | Constraint | Error Code |
|-------|-----------|------------|
| displayName | 1-50 characters, no special chars except spaces and hyphens | INVALID_DISPLAY_NAME |
| dailyCalorieTarget | 500-10000 kcal | INVALID_CALORIE_TARGET |
| macroTargets.protein | 0-500 grams | INVALID_MACRO_TARGET |
| macroTargets.carbs | 0-1000 grams | INVALID_MACRO_TARGET |
| macroTargets.fat | 0-500 grams | INVALID_MACRO_TARGET |
| dietaryPreferences | Max 10 items, each from allowed list | INVALID_DIETARY_PREFERENCE |
| cookingSkillLevel | Must be "beginner", "intermediate", or "advanced" | INVALID_SKILL_LEVEL |
| weightGoal | Must be "lose", "maintain", or "gain" (or null) | INVALID_WEIGHT_GOAL |
| height | 50-300 cm (or null) | INVALID_HEIGHT |
| weight | 20-500 kg (or null) | INVALID_WEIGHT |
| age | 13-120 years (or null) | INVALID_AGE |
| activityLevel | Must be from allowed enum (or null) | INVALID_ACTIVITY_LEVEL |

### Allowed Dietary Preferences
`vegetarian`, `vegan`, `pescatarian`, `keto`, `paleo`, `gluten-free`, `dairy-free`, `nut-free`, `halal`, `kosher`

---

## BR-05: Profile Completion

| Rule | Description |
|------|-------------|
| Minimum required | dailyCalorieTarget must be set to mark profile as complete |
| Onboarding flow | After registration, user is guided through profile setup wizard |
| Skip allowed | User can skip optional fields and complete later |
| profileCompleted flag | Set to `true` when dailyCalorieTarget is set |

---

## BR-06: Offline Sync Rules

| Rule | Description |
|------|-------------|
| Conflict resolution | **Server-wins** — when conflict detected, server version takes priority |
| Sync trigger | Automatic on reconnect (online event detected) |
| Sync order | Oldest mutations first (FIFO queue) |
| Retry on failure | 3 retries with exponential backoff (1s, 2s, 4s) |
| Offline capabilities | View cached data, manual food entry, edit profile |
| Online-only features | Image upload, AI analysis, recipe generation |
| Sync indicator | Show sync status in UI (syncing, synced, error, offline) |
| Data freshness | Cache expires after 24 hours (re-fetch on next online session) |

### Conflict Resolution Detail (Server-Wins)
1. Client sends mutation with `lastKnownVersion` timestamp
2. Server compares with current record's `updatedAt`
3. If `lastKnownVersion < server.updatedAt` → conflict detected
4. Server rejects client mutation, returns current server version
5. Client overwrites local copy with server version
6. User sees toast: "Your changes were overwritten by a newer version"

---

## BR-07: Password Reset

| Rule | Description |
|------|-------------|
| Initiation | User provides email, Cognito sends 6-digit code |
| Code validity | 15 minutes |
| Code attempts | Max 3 attempts before code expires |
| New password | Must meet same password policy (BR-01) |
| Session impact | All existing sessions remain valid (user can optionally sign out all devices) |
