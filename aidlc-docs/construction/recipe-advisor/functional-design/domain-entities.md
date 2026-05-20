# Domain Entities — Recipe Advisor Unit

## Entity: Ingredient (Extended)

Extends the shared `Ingredient` type from `src/shared/types.ts` with inventory-specific fields.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| ingredientId | string (UUID) | Yes | — | Unique identifier (SK in DynamoDB) |
| userId | string | Yes | — | Owner (PK in DynamoDB) |
| name | string | Yes | — | Ingredient name (e.g., "Chicken Breast") |
| category | IngredientCategory | Yes | — | Auto-suggested, user-overridable |
| quantity | number | No | null | Numeric quantity (e.g., 500) |
| unit | QuantityUnit | No | null | Unit of measurement (e.g., "g", "cups") |
| addedAt | string (ISO 8601) | Yes | — | When ingredient was added to inventory |
| updatedAt | string (ISO 8601) | Yes | — | Last modification timestamp |
| source | IngredientSource | Yes | "manual" | How ingredient was added |
| confidence | number | No | null | AI confidence score (0–1), only set when source = "fridge_scan" |

### Sub-Type: IngredientCategory (enum)
```
"protein" | "vegetable" | "fruit" | "dairy" | "grain" | "spice" | "other"
```

### Sub-Type: QuantityUnit (enum)
```
"g" | "kg" | "ml" | "l" | "cups" | "tbsp" | "tsp" | "pieces" | "servings"
```

### Sub-Type: IngredientSource (enum)
```
"manual" | "fridge_scan"
```

---

## Entity: RecipeSuggestion (Transient — not persisted)

Represents an AI-generated recipe suggestion returned from Bedrock. Not stored in DynamoDB unless the user saves it.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| suggestionId | string (UUID) | Yes | Client-generated ID for this suggestion session |
| name | string | Yes | Recipe name |
| description | string | Yes | Short description (1–2 sentences) |
| ingredients | RecipeIngredient[] | Yes | Full ingredient list with quantities |
| steps | string[] | Yes | Ordered cooking instructions |
| nutrition | NutritionData | Yes | Per-serving nutrition (from shared types) |
| prepTime | number | Yes | Preparation time in minutes |
| cookTime | number | Yes | Cooking time in minutes |
| difficulty | DifficultyLevel | Yes | Skill level required |
| servings | number | Yes | Number of servings |
| dietaryTags | string[] | Yes | e.g., ["vegetarian", "gluten-free"] |
| cuisineType | string | No | e.g., "Italian", "Asian" |
| availableIngredients | string[] | Yes | Ingredients from user's inventory used in this recipe |
| missingIngredients | RecipeIngredient[] | Yes | Extra ingredients needed (not in user's inventory) |
| generatedAt | string (ISO 8601) | Yes | When this suggestion was generated |

### Sub-Type: RecipeIngredient
| Field | Type | Description |
|-------|------|-------------|
| name | string | Ingredient name |
| quantity | string | Human-readable quantity (e.g., "200g", "2 cups") |

### Sub-Type: DifficultyLevel (enum)
```
"beginner" | "intermediate" | "advanced"
```

---

## Entity: SavedRecipe (Persisted)

A recipe the user has explicitly saved to their collection. Stored in DynamoDB RecipesTable.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| recipeId | string (UUID) | Yes | — | Unique identifier (SK in DynamoDB) |
| userId | string | Yes | — | Owner (PK in DynamoDB) |
| name | string | Yes | — | Recipe name |
| description | string | Yes | — | Short description |
| ingredients | RecipeIngredient[] | Yes | — | Full ingredient list |
| steps | string[] | Yes | — | Ordered cooking instructions |
| nutrition | NutritionData | Yes | — | Per-serving nutrition |
| prepTime | number | Yes | — | Preparation time in minutes |
| cookTime | number | Yes | — | Cooking time in minutes |
| difficulty | DifficultyLevel | Yes | — | Skill level required |
| servings | number | Yes | — | Number of servings |
| dietaryTags | string[] | Yes | [] | Dietary tags |
| cuisineType | string | No | null | Cuisine type |
| notes | string | No | null | User-editable personal notes |
| savedAt | string (ISO 8601) | Yes | — | When user saved this recipe |
| updatedAt | string (ISO 8601) | Yes | — | Last modification (notes edit) |

---

## Entity: RecipeSuggestionCache (PouchDB — local only)

Caches the last set of recipe suggestions to avoid redundant Bedrock calls.

| Field | Type | Description |
|-------|------|-------------|
| _id | string | PouchDB doc ID: `RECIPE_CACHE#{userId}` |
| suggestions | RecipeSuggestion[] | Cached suggestion list |
| ingredientSnapshot | string[] | Sorted ingredient names at time of generation (for cache invalidation) |
| filterSnapshot | RecipeFilters | Filters used at time of generation |
| generatedAt | string (ISO 8601) | Cache creation timestamp |
| expiresAt | string (ISO 8601) | generatedAt + 14 days |

### Sub-Type: RecipeFilters
| Field | Type | Description |
|-------|------|-------------|
| cookingTime | CookingTimeFilter \| null | Time bucket filter |
| skillLevel | DifficultyLevel \| null | Max skill level |
| cuisineType | string \| null | Cuisine type filter |
| servings | number \| null | Requested servings |

### Sub-Type: CookingTimeFilter (enum)
```
"under_15" | "15_to_30" | "30_to_60" | "over_60"
```

---

## DynamoDB Storage

### IngredientsTable

| PK | SK | Entity |
|----|-----|--------|
| `USER#{userId}` | `ING#{ingredientId}` | Ingredient |

**Access Patterns:**
- Get all ingredients: PK=`USER#{userId}`, SK begins_with `ING#`
- Get single ingredient: PK=`USER#{userId}`, SK=`ING#{ingredientId}`
- Add ingredient: PutItem
- Update ingredient: UpdateItem
- Delete ingredient: DeleteItem
- Bulk add (fridge scan): BatchWriteItem

### RecipesTable

| PK | SK | Entity |
|----|-----|--------|
| `USER#{userId}` | `RECIPE#{recipeId}` | SavedRecipe |

**Access Patterns:**
- Get all saved recipes: PK=`USER#{userId}`, SK begins_with `RECIPE#`
- Get single recipe: PK=`USER#{userId}`, SK=`RECIPE#{recipeId}`
- Save recipe: PutItem (with duplicate name check)
- Delete recipe: DeleteItem

---

## PouchDB Local Storage (Recipe Advisor scope)

| Document | Key | Contents | TTL |
|----------|-----|----------|-----|
| Ingredient inventory | `INGREDIENTS#{userId}` | Full ingredient list snapshot | 24h (re-fetch on reconnect) |
| Saved recipes | `SAVED_RECIPES#{userId}` | Full saved recipe list | 24h (re-fetch on reconnect) |
| Recipe suggestion cache | `RECIPE_CACHE#{userId}` | Last suggestion set + metadata | 14 days or until ingredients change |
| Offline mutation queue | `SYNC_QUEUE#{userId}` | Pending ingredient add/update/delete | Until synced |
