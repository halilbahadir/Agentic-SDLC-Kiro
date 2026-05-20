# Component Methods

## Backend Service Methods

### BC-01: Auth Service

| Method | Input | Output | Purpose |
|--------|-------|--------|---------|
| `registerUser` | `{ email, password }` | `{ userId, confirmationRequired }` | Create new user in Cognito |
| `confirmRegistration` | `{ email, code }` | `{ success }` | Verify email with confirmation code |
| `loginUser` | `{ email, password }` | `{ accessToken, refreshToken, idToken }` | Authenticate and return tokens |
| `refreshToken` | `{ refreshToken }` | `{ accessToken, idToken }` | Refresh expired access token |
| `forgotPassword` | `{ email }` | `{ deliveryMedium }` | Initiate password reset |
| `confirmForgotPassword` | `{ email, code, newPassword }` | `{ success }` | Complete password reset |
| `getProfile` | `{ userId }` | `{ profile }` | Get user profile and preferences |
| `updateProfile` | `{ userId, updates }` | `{ profile }` | Update profile (goals, preferences, skill) |

### BC-02: Meal Service

| Method | Input | Output | Purpose |
|--------|-------|--------|---------|
| `createMeal` | `{ userId, mealType, items[], date }` | `{ mealId, nutrition }` | Log a new meal with food items |
| `getMeal` | `{ userId, mealId }` | `{ meal }` | Get single meal details |
| `updateMeal` | `{ userId, mealId, updates }` | `{ meal }` | Edit meal items or metadata |
| `deleteMeal` | `{ userId, mealId }` | `{ success }` | Remove a meal entry |
| `getMealHistory` | `{ userId, startDate, endDate, limit, cursor }` | `{ meals[], nextCursor }` | Paginated meal history |
| `getDailySummary` | `{ userId, date }` | `{ totalCalories, macros, meals[] }` | Aggregated daily nutrition |
| `getWeeklySummary` | `{ userId, weekStart }` | `{ dailySummaries[], averages }` | Weekly nutrition overview |
| `searchFoods` | `{ query, limit }` | `{ foods[] }` | Search food database |
| `createCustomFood` | `{ userId, name, nutrition }` | `{ foodId }` | Add custom food to user's database |

### BC-03: AI Analysis Service

| Method | Input | Output | Purpose |
|--------|-------|--------|---------|
| `analyzeMealImage` | `{ imageKey, userId }` | `{ items[]{name, portion, confidence, nutrition} }` | Identify foods in meal photo |
| `analyzeFridgeImage` | `{ imageKey, userId }` | `{ ingredients[]{name, category, confidence} }` | Identify ingredients in fridge photo |
| `estimateNutrition` | `{ foodName, portionSize, portionUnit }` | `{ nutrition }` | Estimate nutrition for a food item |

### BC-04: Recipe Service

| Method | Input | Output | Purpose |
|--------|-------|--------|---------|
| `suggestRecipes` | `{ userId, ingredients[], filters }` | `{ recipes[] }` | Generate recipe suggestions via AI |
| `getRecipeDetails` | `{ recipeId }` | `{ recipe }` | Get full recipe with steps and nutrition |
| `saveRecipe` | `{ userId, recipe }` | `{ recipeId }` | Save a recipe to user's collection |
| `getSavedRecipes` | `{ userId, limit, cursor }` | `{ recipes[], nextCursor }` | List user's saved recipes |
| `deleteSavedRecipe` | `{ userId, recipeId }` | `{ success }` | Remove from saved recipes |

### BC-05: Ingredient Service

| Method | Input | Output | Purpose |
|--------|-------|--------|---------|
| `addIngredient` | `{ userId, name, category, quantity }` | `{ ingredientId }` | Add ingredient to inventory |
| `removeIngredient` | `{ userId, ingredientId }` | `{ success }` | Remove from inventory |
| `updateIngredient` | `{ userId, ingredientId, updates }` | `{ ingredient }` | Update quantity or details |
| `getIngredients` | `{ userId }` | `{ ingredients[] }` | Get full ingredient inventory |
| `bulkAddIngredients` | `{ userId, ingredients[] }` | `{ ingredientIds[] }` | Add multiple (from AI scan) |
| `searchIngredients` | `{ query }` | `{ suggestions[] }` | Autocomplete for ingredient names |

### BC-06: Image Service

| Method | Input | Output | Purpose |
|--------|-------|--------|---------|
| `getUploadUrl` | `{ userId, imageType, contentType }` | `{ uploadUrl, imageKey }` | Generate pre-signed S3 upload URL |
| `getImageUrl` | `{ imageKey }` | `{ url }` | Get signed URL for image retrieval |
| `deleteImage` | `{ imageKey }` | `{ success }` | Remove image from S3 |

---

## Frontend Hook Methods

### Auth Hooks

| Hook | Returns | Purpose |
|------|---------|---------|
| `useAuth()` | `{ user, login, logout, register, isAuthenticated }` | Auth state and actions |
| `useProfile()` | `{ profile, updateProfile, isLoading }` | User profile management |

### Meal Hooks

| Hook | Returns | Purpose |
|------|---------|---------|
| `useMeals(date)` | `{ meals, isLoading, error }` | Fetch meals for a date |
| `useCreateMeal()` | `{ createMeal, isLoading }` | Create new meal mutation |
| `useDailySummary(date)` | `{ summary, isLoading }` | Daily nutrition summary |
| `useWeeklySummary(week)` | `{ summary, isLoading }` | Weekly nutrition summary |
| `useFoodSearch(query)` | `{ results, isLoading }` | Food database search |

### AI Hooks

| Hook | Returns | Purpose |
|------|---------|---------|
| `useImageAnalysis()` | `{ analyze, results, isAnalyzing }` | Trigger and track AI analysis |
| `useRecipeSuggestions(filters)` | `{ recipes, suggest, isLoading }` | Recipe generation |

### Ingredient Hooks

| Hook | Returns | Purpose |
|------|---------|---------|
| `useIngredients()` | `{ ingredients, add, remove, update, isLoading }` | Ingredient CRUD |

### Sync Hooks

| Hook | Returns | Purpose |
|------|---------|---------|
| `useOfflineSync()` | `{ isOnline, syncStatus, pendingChanges }` | Sync state |
