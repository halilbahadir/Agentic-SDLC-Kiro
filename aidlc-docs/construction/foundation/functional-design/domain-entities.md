# Domain Entities — Foundation Unit

## Entity: User

Represents an authenticated user in the system (managed by Cognito).

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| userId | string (UUID) | Yes | Cognito sub (unique identifier) |
| email | string | Yes | User's email address (unique, used for login) |
| emailVerified | boolean | Yes | Whether email has been confirmed |
| createdAt | string (ISO 8601) | Yes | Account creation timestamp |
| lastLoginAt | string (ISO 8601) | No | Last successful login timestamp |

**Notes**: User entity is managed by Cognito. Only `userId` and `email` are stored in DynamoDB (as part of UserProfile).

---

## Entity: UserProfile

Represents user preferences, goals, and personal data stored in DynamoDB.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| userId | string | Yes | — | Partition key, matches Cognito sub |
| email | string | Yes | — | Denormalized from Cognito for display |
| displayName | string | No | null | User's display name |
| timezone | string | No | "UTC" | IANA timezone (e.g., "Europe/London") |
| dailyCalorieTarget | number | No | 2000 | Daily calorie goal (kcal) |
| macroTargets | MacroTargets | No | default | Daily macronutrient targets |
| dietaryPreferences | string[] | No | [] | e.g., ["vegetarian", "gluten-free"] |
| cookingSkillLevel | enum | No | "beginner" | "beginner" / "intermediate" / "advanced" |
| weightGoal | enum | No | null | "lose" / "maintain" / "gain" |
| height | number | No | null | Height in cm |
| weight | number | No | null | Weight in kg |
| age | number | No | null | Age in years |
| activityLevel | enum | No | null | "sedentary" / "light" / "moderate" / "active" / "very_active" |
| profileCompleted | boolean | Yes | false | Whether onboarding wizard is complete |
| createdAt | string (ISO 8601) | Yes | — | Profile creation timestamp |
| updatedAt | string (ISO 8601) | Yes | — | Last profile update timestamp |

### Sub-Type: MacroTargets

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| protein | number | 50 | Daily protein target (grams) |
| carbs | number | 250 | Daily carbohydrate target (grams) |
| fat | number | 65 | Daily fat target (grams) |

---

## Entity: SyncRecord

Represents a queued mutation for offline sync (stored in PouchDB locally).

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| _id | string | Yes | PouchDB document ID |
| _rev | string | Yes | PouchDB revision (for conflict detection) |
| entityType | string | Yes | "meal" / "ingredient" / "recipe" / "profile" |
| entityId | string | Yes | ID of the entity being modified |
| operation | enum | Yes | "create" / "update" / "delete" |
| payload | object | Yes | The mutation data |
| timestamp | string (ISO 8601) | Yes | When the mutation was made |
| synced | boolean | Yes | Whether this has been pushed to server |
| userId | string | Yes | Owner of this record |

---

## Entity: SyncState

Represents the overall sync status (stored in PouchDB locally).

| Field | Type | Description |
|-------|------|-------------|
| lastSyncTimestamp | string (ISO 8601) | Last successful sync with server |
| pendingCount | number | Number of unsynced mutations |
| syncStatus | enum | "idle" / "syncing" / "error" / "offline" |
| lastError | string | null | Last sync error message |

---

## DynamoDB Storage (Foundation)

### MealsNutrition Table — UserProfile entries

| PK | SK | Entity |
|----|-----|--------|
| `USER#{userId}` | `PROFILE#` | UserProfile |

**Access Patterns:**
- Get profile: PK=`USER#{userId}`, SK=`PROFILE#`
- Update profile: Same key, update expression
