# Service Layer Design

## Service Architecture

The backend follows a **service-per-feature** pattern with separate Lambda functions per feature area, orchestrated through API Gateway.

```
+------------------+     +------------------+     +------------------+
|   CloudFront     | --> |   API Gateway    | --> |  Lambda Functions |
|   (Frontend)     |     |   (REST Router)  |     |  (per-feature)   |
+------------------+     +------------------+     +------------------+
                                |                    |    |    |
                                v                    v    v    v
                         +-------------+   +---------+ +---+ +-------+
                         |   Cognito   |   |DynamoDB | | S3| |Bedrock|
                         | (Authorizer)|   |(Storage)| |   | |(AI/ML)|
                         +-------------+   +---------+ +---+ +-------+
```

---

## Service Definitions

### Auth Service (Lambda: `calorie-app-auth`)
**Trigger**: API Gateway `/auth/*` routes  
**Runtime**: Node.js 20.x (TypeScript)  
**Responsibilities**:
- Proxy registration/login to Cognito
- Manage user profile in DynamoDB
- Token validation for protected routes

**Interactions**:
- Cognito User Pool (auth operations)
- DynamoDB UserProfiles table (profile CRUD)

---

### Meal Service (Lambda: `calorie-app-meals`)
**Trigger**: API Gateway `/meals/*` routes  
**Runtime**: Node.js 20.x (TypeScript)  
**Responsibilities**:
- CRUD operations for meal entries
- Nutrition aggregation and summaries
- Food database search
- Custom food management

**Interactions**:
- DynamoDB MealsNutrition table (meal CRUD, daily/weekly queries)
- DynamoDB CustomFoods table (user's custom foods)

---

### AI Analysis Service (Lambda: `calorie-app-ai`)
**Trigger**: API Gateway `/ai/*` routes  
**Runtime**: Node.js 20.x (TypeScript)  
**Responsibilities**:
- Orchestrate Bedrock calls for image analysis
- Parse and structure AI responses
- Nutrition estimation from food identification

**Interactions**:
- AWS Bedrock (Claude/Nova for multimodal analysis)
- S3 (read uploaded images)
- DynamoDB MealsNutrition table (store analysis results)

---

### Recipe Service (Lambda: `calorie-app-recipes`)
**Trigger**: API Gateway `/recipes/*` routes  
**Runtime**: Node.js 20.x (TypeScript)  
**Responsibilities**:
- Generate recipes via Bedrock based on ingredients and preferences
- Manage saved recipes
- Calculate recipe nutrition

**Interactions**:
- AWS Bedrock (recipe generation prompts)
- DynamoDB Recipes table (saved recipes)
- Ingredient Service (get available ingredients)

---

### Ingredient Service (Lambda: `calorie-app-ingredients`)
**Trigger**: API Gateway `/ingredients/*` routes  
**Runtime**: Node.js 20.x (TypeScript)  
**Responsibilities**:
- Ingredient inventory CRUD
- Categorization logic
- Bulk add from AI scan results

**Interactions**:
- DynamoDB Ingredients table (inventory CRUD)

---

### Image Service (Lambda: `calorie-app-images`)
**Trigger**: API Gateway `/images/*` routes  
**Runtime**: Node.js 20.x (TypeScript)  
**Responsibilities**:
- Generate pre-signed S3 upload URLs
- Generate signed retrieval URLs
- Image lifecycle management

**Interactions**:
- S3 bucket (pre-signed URL generation)

---

## Service Orchestration Patterns

### Meal Logging with Photo (Multi-Service Flow)
```
1. Frontend → Image Service: getUploadUrl()
2. Frontend → S3: Upload image (pre-signed URL)
3. Frontend → AI Analysis Service: analyzeMealImage(imageKey)
4. AI Analysis Service → S3: Read image
5. AI Analysis Service → Bedrock: Analyze image
6. AI Analysis Service → Frontend: Return identified items
7. Frontend: User confirms/corrects items
8. Frontend → Meal Service: createMeal(confirmedItems)
```

### Recipe Suggestion Flow
```
1. Frontend → Ingredient Service: getIngredients()
2. Frontend → Recipe Service: suggestRecipes(ingredients, filters)
3. Recipe Service → Bedrock: Generate recipes
4. Recipe Service → Frontend: Return recipe suggestions
```

### Offline Sync Flow
```
1. Frontend (offline): Store mutations in PouchDB
2. Frontend (reconnect): Detect online status
3. Frontend: Replay queued mutations to respective services
4. Services: Process mutations, return results
5. Frontend: Update local PouchDB with server responses
```

---

## Cross-Cutting Concerns

### Error Handling Pattern
All services follow consistent error response format:
```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Meal not found",
    "details": {}
  }
}
```

### Logging Pattern
All services use structured JSON logging:
```json
{
  "level": "INFO",
  "service": "meal-service",
  "method": "createMeal",
  "userId": "user-123",
  "timestamp": "2026-05-17T00:00:00Z",
  "duration": 150
}
```

### Authentication Pattern
All protected routes use Cognito JWT authorizer at API Gateway level. Lambda functions receive validated user context in the event object.
