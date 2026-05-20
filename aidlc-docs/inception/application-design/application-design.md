# Application Design — Consolidated

## Architecture Summary

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Backend Pattern | Service-per-feature (Lambda per feature) | Balance between simplicity and separation of concerns |
| API Style | REST with resource-based URLs | Simple, well-understood, good tooling support |
| Frontend State | React Context + useReducer | Built-in, no extra dependencies, sufficient for this app |
| Authentication | AWS Cognito | Managed service, handles email/password, tokens, password reset |
| Image Pipeline | Direct to Bedrock (S3 → Lambda → Bedrock) | Simplest path, no pre-processing needed initially |
| Offline Sync | PouchDB/CouchDB pattern | Built-in sync protocol with conflict resolution |
| API Communication | axios/fetch with custom hooks | Simple, familiar, easy to test |
| Database Design | Hybrid DynamoDB | Single table for related entities, separate for independent |

---

## System Architecture

```
+------------------------------------------------------------------+
|                         FRONTEND (React + TypeScript)            |
|                                                                  |
|  +----------+  +----------+  +----------+  +----------+          |
|  |   Auth   |  |   Meal   |  | Nutrition|  |  Recipe  |          |
|  |  Module  |  | Logging  |  | Dashboard|  | Advisor  |          |
|  +----------+  +----------+  +----------+  +----------+          |
|  +----------+  +----------+  +----------+                        |
|  |Ingredient|  | Offline  |  | Shared   |                        |
|  |  Mgmt    |  |  Sync    |  |   UI     |                        |
|  +----------+  +----------+  +----------+                        |
|       |              |              |                            |
|       v              v              v                            |
|  +------------------+  +------------------+                      |
|  | axios/fetch REST |  |    PouchDB       |                      |
|  |   (custom hooks) |  | (local storage)  |                      |
|  +------------------+  +------------------+                      |
+------------------------------------------------------------------+
            |                        |
            v                        v (sync on reconnect)
+------------------------------------------------------------------+
|                    AWS CLOUD INFRASTRUCTURE                      |
|                                                                  |
|  +------------------+     +------------------+                   |
|  |   CloudFront     |     |   API Gateway    |                   |
|  |   (CDN + SPA)    |     |   (REST Router)  |                   |
|  +------------------+     +------------------+                   |
|                                   |                              |
|                    +--------------+--------------+               |
|                    |              |              |               |
|                    v              v              v               |
|  +----------+ +----------+ +----------+ +----------+             |
|  |   Auth   | |   Meal   | |    AI    | |  Recipe  |             |
|  | Service  | | Service  | | Analysis | | Service  |             |
|  | (Lambda) | | (Lambda) | | (Lambda) | | (Lambda) |             |
|  +----------+ +----------+ +----------+ +----------+             |
|  +----------+ +----------+                                       |
|  |Ingredient| |  Image   |                                       |
|  | Service  | | Service  |                                       |
|  | (Lambda) | | (Lambda) |                                       |
|  +----------+ +----------+                                       |
|       |              |              |              |             |
|       v              v              v              v             |
|  +----------+  +----------+  +----------+  +----------+          |
|  | Cognito  |  | DynamoDB |  |    S3    |  |  Bedrock |          |
|  |(User Pool)|  | (Tables) |  | (Images) |  |  (AI/ML) |         |
|  +----------+  +----------+  +----------+  +----------+          |
+------------------------------------------------------------------+
```

---

## DynamoDB Table Design (Hybrid)

### Table 1: MealsNutrition (Single-table for related entities)
**Purpose**: Meals, meal items, daily summaries, custom foods — all user-meal-related data

| PK | SK | Entity | Data |
|----|-----|--------|------|
| `USER#userId` | `MEAL#date#mealId` | Meal | mealType, items, totalNutrition, imageKey |
| `USER#userId` | `DAILY#date` | DailySummary | totalCalories, macros, mealCount |
| `USER#userId` | `FOOD#foodId` | CustomFood | name, nutrition, createdAt |
| `USER#userId` | `PROFILE#` | UserProfile | goals, preferences, skillLevel |

**GSI-1**: `GSI1PK=USER#userId`, `GSI1SK=date` (for date-range queries)

### Table 2: Ingredients (Separate — independent domain)
**Purpose**: User ingredient inventories

| PK | SK | Data |
|----|-----|------|
| `USER#userId` | `ING#ingredientId` | name, category, quantity, addedAt |

### Table 3: Recipes (Separate — independent domain)
**Purpose**: Saved recipes

| PK | SK | Data |
|----|-----|------|
| `USER#userId` | `RECIPE#recipeId` | name, ingredients, steps, nutrition, metadata |

---

## API Routes

### Auth Routes (`/auth`)
| Method | Path | Handler | Auth |
|--------|------|---------|------|
| POST | `/auth/register` | registerUser | Public |
| POST | `/auth/confirm` | confirmRegistration | Public |
| POST | `/auth/login` | loginUser | Public |
| POST | `/auth/refresh` | refreshToken | Public |
| POST | `/auth/forgot-password` | forgotPassword | Public |
| POST | `/auth/confirm-password` | confirmForgotPassword | Public |
| GET | `/auth/profile` | getProfile | Protected |
| PUT | `/auth/profile` | updateProfile | Protected |

### Meal Routes (`/meals`)
| Method | Path | Handler | Auth |
|--------|------|---------|------|
| POST | `/meals` | createMeal | Protected |
| GET | `/meals/:mealId` | getMeal | Protected |
| PUT | `/meals/:mealId` | updateMeal | Protected |
| DELETE | `/meals/:mealId` | deleteMeal | Protected |
| GET | `/meals/history` | getMealHistory | Protected |
| GET | `/meals/summary/daily/:date` | getDailySummary | Protected |
| GET | `/meals/summary/weekly/:weekStart` | getWeeklySummary | Protected |
| GET | `/meals/foods/search` | searchFoods | Protected |
| POST | `/meals/foods/custom` | createCustomFood | Protected |

### AI Routes (`/ai`)
| Method | Path | Handler | Auth |
|--------|------|---------|------|
| POST | `/ai/analyze/meal` | analyzeMealImage | Protected |
| POST | `/ai/analyze/fridge` | analyzeFridgeImage | Protected |
| POST | `/ai/estimate-nutrition` | estimateNutrition | Protected |

### Recipe Routes (`/recipes`)
| Method | Path | Handler | Auth |
|--------|------|---------|------|
| POST | `/recipes/suggest` | suggestRecipes | Protected |
| GET | `/recipes/:recipeId` | getRecipeDetails | Protected |
| POST | `/recipes/saved` | saveRecipe | Protected |
| GET | `/recipes/saved` | getSavedRecipes | Protected |
| DELETE | `/recipes/saved/:recipeId` | deleteSavedRecipe | Protected |

### Ingredient Routes (`/ingredients`)
| Method | Path | Handler | Auth |
|--------|------|---------|------|
| POST | `/ingredients` | addIngredient | Protected |
| GET | `/ingredients` | getIngredients | Protected |
| PUT | `/ingredients/:ingredientId` | updateIngredient | Protected |
| DELETE | `/ingredients/:ingredientId` | removeIngredient | Protected |
| POST | `/ingredients/bulk` | bulkAddIngredients | Protected |
| GET | `/ingredients/search` | searchIngredients | Protected |

### Image Routes (`/images`)
| Method | Path | Handler | Auth |
|--------|------|---------|------|
| POST | `/images/upload-url` | getUploadUrl | Protected |
| GET | `/images/:imageKey` | getImageUrl | Protected |
| DELETE | `/images/:imageKey` | deleteImage | Protected |

---

## Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Frontend Framework | React | 18.x |
| Frontend Language | TypeScript | 5.x |
| Build Tool | Vite | 5.x |
| UI Styling | Tailwind CSS | 3.x |
| Charts | Recharts | 2.x |
| HTTP Client | axios | 1.x |
| Offline Storage | PouchDB | 8.x |
| Backend Runtime | Node.js | 20.x |
| Backend Language | TypeScript | 5.x |
| IaC | AWS CDK | 2.x (TypeScript) |
| Database | DynamoDB | - |
| Object Storage | S3 | - |
| AI/ML | AWS Bedrock (Claude 3.5 Sonnet) | - |
| Auth | AWS Cognito | - |
| API | API Gateway (REST) | - |
| CDN | CloudFront | - |
| Hosting | S3 + CloudFront (SPA) | - |

---

## Design Principles (Inherited by all units)

1. **Service Independence**: Each Lambda function is self-contained with its own dependencies
2. **Shared Types**: Common TypeScript interfaces shared via a `shared/types` package
3. **REST Conventions**: Standard HTTP methods, status codes, error format across all services
4. **Auth at Gateway**: Cognito JWT validation at API Gateway level, not in Lambda code
5. **Structured Logging**: JSON logs with service name, method, userId, timestamp, duration
6. **Error Format**: Consistent `{ error: { code, message, details } }` across all services
7. **Offline-First**: Frontend assumes offline capability; online is a bonus
8. **User Isolation**: All data access scoped by userId (partition key)
