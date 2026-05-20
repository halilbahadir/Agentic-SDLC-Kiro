# Unit of Work Dependencies

## Dependency Matrix

| Unit | Depends On | Provides To | Dependency Type |
|------|-----------|-------------|-----------------|
| Foundation | None | Calorie Tracking, Recipe Advisor | Infrastructure, Auth, Shared Types |
| Calorie Tracking | Foundation | Recipe Advisor | AI Analysis Service (fridge photo endpoint) |
| Recipe Advisor | Foundation, Calorie Tracking (partial) | None | Uses AI Analysis for fridge scanning |

## Dependency Diagram

```
+------------------+
|   Foundation     |  (Merge Order: 1)
|   (4 stories)   |
+------------------+
       |         |
       v         v
+------------+  +---------------+
|  Calorie   |  |    Recipe     |  (Merge Order: 2 — parallel)
|  Tracking  |  |    Advisor    |
| (8 stories)|  |  (5 stories)  |
+------------+  +---------------+
       |                ^
       |                |
       +--- contract ---+
       (POST /ai/analyze/fridge)
```

## Dependency Details

### Foundation → Calorie Tracking
| What Foundation Provides | Used By |
|--------------------------|---------|
| Cognito User Pool + tokens | All authenticated API calls |
| DynamoDB MealsNutrition table | Meal CRUD, daily summaries |
| DynamoDB CustomFoods (in MealsNutrition) | Custom food items |
| S3 bucket (images) | Meal photo uploads |
| API Gateway + Cognito authorizer | All REST endpoints |
| Shared types (`NutritionData`, `MealItem`) | Backend + frontend |
| Auth module (login/register) | User must be authenticated |
| Offline Sync module (PouchDB) | Offline meal logging |
| Shared UI components | Meal Logging and Dashboard UI |

### Foundation → Recipe Advisor
| What Foundation Provides | Used By |
|--------------------------|---------|
| Cognito User Pool + tokens | All authenticated API calls |
| DynamoDB Ingredients table | Ingredient CRUD |
| DynamoDB Recipes table | Saved recipes |
| S3 bucket (images) | Fridge photo uploads |
| API Gateway + Cognito authorizer | All REST endpoints |
| Shared types (`Ingredient`, `Recipe`) | Backend + frontend |
| Auth module (login/register) | User must be authenticated |
| Offline Sync module (PouchDB) | Offline ingredient list |
| Shared UI components | Ingredient and Recipe UI |

### Calorie Tracking → Recipe Advisor (Cross-Unit Contract)
| What Calorie Tracking Provides | Used By |
|-------------------------------|---------|
| `POST /ai/analyze/fridge` endpoint | Fridge photo ingredient detection |

**Contract Definition:**
```typescript
// Endpoint: POST /ai/analyze/fridge
// Owner: AI Analysis Service (Calorie Tracking unit)
// Consumer: Recipe Advisor unit (Ingredient Management module)

interface AnalyzeFridgeRequest {
  imageKey: string;  // S3 key of uploaded fridge image
  userId: string;    // Authenticated user ID
}

interface AnalyzeFridgeResponse {
  ingredients: {
    name: string;
    category: 'protein' | 'vegetable' | 'fruit' | 'dairy' | 'grain' | 'spice' | 'other';
    confidence: number;  // 0-1
  }[];
}
```

## Parallelism Analysis

| Scenario | Calorie Tracking | Recipe Advisor |
|----------|-----------------|----------------|
| Foundation complete | ✅ Can start immediately | ✅ Can start immediately |
| Fridge photo feature (US-11) | Provides endpoint | Needs contract only (can mock) |
| All other Recipe features | Independent | Independent |

**Conclusion**: After Foundation is complete, both Calorie Tracking and Recipe Advisor can proceed in parallel. The fridge photo contract is simple enough that Recipe Advisor can mock it during development and integrate when Calorie Tracking delivers the endpoint.

## Merge Order

| Order | Unit | Rationale |
|-------|------|-----------|
| 1 | Foundation | No dependencies, provides infrastructure for all other units |
| 2 | Calorie Tracking | Depends on Foundation only, provides contract to Recipe Advisor |
| 2 | Recipe Advisor | Depends on Foundation, can merge in parallel with Calorie Tracking (contract tests validate fridge endpoint) |

**Note**: Calorie Tracking and Recipe Advisor are both Order 2. They can merge in any sequence since the cross-unit dependency (fridge photo endpoint) is covered by a contract test.
