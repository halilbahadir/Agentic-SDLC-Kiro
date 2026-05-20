# Component Dependencies

## Dependency Matrix

### Backend Service Dependencies

| Service | Depends On | Communication | Data Shared |
|---------|-----------|---------------|-------------|
| Auth Service | Cognito, DynamoDB (UserProfiles) | AWS SDK | User tokens, profile data |
| Meal Service | DynamoDB (MealsNutrition, CustomFoods) | AWS SDK | Meal records, nutrition data |
| AI Analysis Service | S3, Bedrock, DynamoDB | AWS SDK | Images, AI responses |
| Recipe Service | Bedrock, DynamoDB (Recipes) | AWS SDK | Recipe data, AI responses |
| Ingredient Service | DynamoDB (Ingredients) | AWS SDK | Ingredient inventory |
| Image Service | S3 | AWS SDK | Image metadata, URLs |

### Frontend Module Dependencies

| Module | Depends On | Communication |
|--------|-----------|---------------|
| Auth Module | Auth Service | REST (axios) |
| Meal Logging Module | Meal Service, AI Analysis Service, Image Service | REST (axios) |
| Nutrition Dashboard | Meal Service | REST (axios) |
| Ingredient Management | Ingredient Service, AI Analysis Service, Image Service | REST (axios) |
| Recipe Advisor | Recipe Service, Ingredient Service | REST (axios) |
| Offline Sync | PouchDB, All Services (for replay) | PouchDB sync + REST |
| Shared UI | None (leaf dependency) | N/A |

---

## Communication Patterns

### Frontend → Backend
```
+-------------------+        +------------------+        +------------------+
|   React App       | -----> |   API Gateway    | -----> |   Lambda         |
|   (axios/fetch)   |  HTTPS |   (REST)         |  Event |   (per-feature)  |
+-------------------+        +------------------+        +------------------+
        |                            |
        |                            v
        |                    +------------------+
        |                    |   Cognito        |
        |                    |   (JWT verify)   |
        |                    +------------------+
        |
        v
+-------------------+
|   PouchDB         |
|   (local cache)   |
+-------------------+
```

### Backend → AWS Services
```
+------------------+        +------------------+
|   Lambda         | -----> |   DynamoDB       |
|   (service)      |  SDK   |   (data store)   |
+------------------+        +------------------+
        |
        |-----> S3 (images)
        |
        |-----> Bedrock (AI/ML)
        |
        |-----> Cognito (auth ops)
```

---

## Data Flow Diagrams

### Meal Logging Flow (Photo)
```
User Camera/Gallery
       |
       v
[Image Upload] --> S3 Bucket --> [AI Analysis Lambda] --> Bedrock
       |                                    |
       v                                    v
[Confirmation UI] <--- Identified Items + Nutrition
       |
       v
[Save Meal] --> Meal Service Lambda --> DynamoDB (MealsNutrition)
       |
       v
[Update Dashboard] <--- Daily Summary
```

### Recipe Discovery Flow
```
[Ingredient List] --> Ingredient Service --> DynamoDB (Ingredients)
       |
       v
[Request Recipes] --> Recipe Service Lambda --> Bedrock (AI Generation)
       |                                           |
       v                                           v
[Display Recipes] <--- Generated Recipes with Nutrition
       |
       v
[Save Recipe] --> Recipe Service --> DynamoDB (Recipes)
```

### Offline Sync Flow
```
[User Action (offline)]
       |
       v
[PouchDB Local Store] --- queue mutation --->
       |                                      |
       v                                      v
[Show cached data]              [On Reconnect: Replay to API]
                                              |
                                              v
                                [API Services process mutations]
                                              |
                                              v
                                [PouchDB syncs with server state]
```

---

## Dependency Rules

1. **Frontend modules** depend on backend services via REST only (no direct AWS SDK calls from frontend except S3 pre-signed uploads)
2. **Backend services** are independent — no Lambda-to-Lambda calls. Each service owns its data.
3. **Shared data** is accessed through API calls, not shared database access between services
4. **AI Analysis Service** is called by frontend directly (not by other backend services)
5. **Recipe Service** reads ingredient data by querying DynamoDB directly (same account, shared table access for read efficiency)
6. **Offline Sync** uses PouchDB locally and replays REST calls on reconnect — no special sync endpoint needed

---

## Shared Contracts

### Common Types (shared between frontend and backend)

```typescript
// Nutrition data structure (used across all services)
interface NutritionData {
  calories: number;
  protein: number;
  carbs: number;
  fat: number;
  fiber: number;
  vitamins: { a: number; c: number; d: number; b12: number };
  minerals: { iron: number; calcium: number; potassium: number; sodium: number };
}

// Meal item (used by Meal Service and AI Analysis)
interface MealItem {
  name: string;
  portion: number;
  portionUnit: 'g' | 'cups' | 'pieces' | 'servings';
  nutrition: NutritionData;
  confidence?: number; // AI confidence score
}

// Ingredient (used by Ingredient Service and Recipe Service)
interface Ingredient {
  id: string;
  name: string;
  category: 'protein' | 'vegetable' | 'fruit' | 'dairy' | 'grain' | 'spice' | 'other';
  quantity?: string;
}

// Recipe (used by Recipe Service)
interface Recipe {
  id: string;
  name: string;
  description: string;
  ingredients: { name: string; quantity: string }[];
  steps: string[];
  nutrition: NutritionData;
  prepTime: number; // minutes
  cookTime: number; // minutes
  difficulty: 'beginner' | 'intermediate' | 'advanced';
  servings: number;
  dietaryTags: string[];
}
```
