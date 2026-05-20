# Business Logic Model — Recipe Advisor Unit

## 1. Ingredient Management Flows

### Add Ingredient Manually (US-12)
```
1. User taps FAB on IngredientListPage
2. AddIngredientModal opens
3. User types ingredient name
4. Frontend looks up name in category lookup table → suggests category
5. User reviews/overrides category, enters quantity + unit (optional)
6. Frontend validates per BR-RA-01
7. If online:
   a. POST /ingredients { name, category, quantity, unit }
   b. Ingredient Service validates server-side (BR-RA-01)
   c. Ingredient Service → DynamoDB: PutItem (IngredientsTable)
   d. Return { ingredientId, ...ingredient }
   e. Frontend updates local ingredient list + PouchDB cache
8. If offline:
   a. Generate client-side UUID for ingredientId
   b. Save to PouchDB with synced=false
   c. Update local UI optimistically
   d. On reconnect: sync queue replays POST /ingredients
```

### Remove Ingredient (US-12)
```
1. User swipe-deletes or long-presses ingredient on IngredientListPage
2. Confirmation: "Remove [name] from your inventory?"
3. If online:
   a. DELETE /ingredients/{ingredientId}
   b. Ingredient Service → DynamoDB: DeleteItem
   c. Frontend removes from local list + PouchDB cache
   d. Invalidate recipe suggestion cache (BR-RA-04)
4. If offline:
   a. Mark ingredient as deleted in PouchDB (synced=false, operation="delete")
   b. Remove from local UI immediately
   c. On reconnect: sync queue replays DELETE /ingredients/{ingredientId}
   d. After sync: invalidate recipe suggestion cache
```

### Update Ingredient (US-12)
```
1. User taps ingredient to edit quantity/unit/category
2. EditIngredientModal opens with current values
3. User modifies fields, frontend validates
4. If online:
   a. PUT /ingredients/{ingredientId} { updates }
   b. Ingredient Service → DynamoDB: UpdateItem
   c. Frontend updates local list + PouchDB cache
   d. Invalidate recipe suggestion cache
5. If offline: same queue pattern as Add
```

---

## 2. Fridge Scan Flow (US-11)

```
1. User taps "Scan Fridge" button on IngredientListPage
2. ImageUpload component opens (camera or gallery)
3. User selects/captures fridge photo
4. Frontend validates: size ≤ 10MB, format JPEG/PNG/WEBP (BR-RA-03)
5. Frontend checks online status:
   - If offline: show "Fridge scan requires internet connection" → stop
6. Frontend → POST /images/upload-url { imageType: "fridge", contentType }
7. Image Service returns { uploadUrl, imageKey }
8. Frontend → S3: PUT image (pre-signed URL)
9. Frontend → POST /ai/analyze/fridge { imageKey, userId }
   [Note: This endpoint is owned by AI Analysis Service / Calorie Tracking unit]
10. AI Analysis Service → S3: read image
11. AI Analysis Service → Bedrock: analyze fridge image
12. Return { ingredients[]: { name, category, confidence } }
13. Frontend filters: remove items with confidence < 0.3 (BR-RA-03)
14. If 0 items remain: show "No ingredients could be identified" message → stop
15. FridgeScanResultsView opens:
    - All items above threshold shown pre-selected
    - Each item shows: name (editable), category (editable), confidence badge
    - User deselects unwanted items, edits names/categories
16. User taps "Add to Inventory"
17. Frontend deduplicates against existing inventory (case-insensitive name match)
18. Frontend → POST /ingredients/bulk { ingredients: [confirmed items] }
19. Ingredient Service → DynamoDB: BatchWriteItem (IngredientsTable)
20. Frontend updates local list + PouchDB cache
21. Invalidate recipe suggestion cache
```

---

## 3. Recipe Suggestion Flow (US-13)

### Initial Suggestion Request
```
1. User navigates to RecipeAdvisorPage
2. User optionally sets filters (cooking time, cuisine type, servings)
   - Skill level and dietary prefs pre-populated from profile
3. User taps "Get Recipes"
4. Frontend checks online status:
   - If offline: show "Recipe generation requires internet connection" → stop
5. Frontend checks suggestion cache (PouchDB RECIPE_CACHE#{userId}):
   a. If cache exists AND not expired (< 14 days) AND ingredient list unchanged AND filters unchanged:
      → Return cached suggestions (skip steps 6–10)
   b. Otherwise: proceed to Bedrock call
6. Frontend collects:
   - Current ingredient list (from local state / PouchDB)
   - User profile: dietaryPreferences, cookingSkillLevel, dailyCalorieTarget
   - Active filters: cookingTime, cuisineType, servings
7. Frontend → POST /recipes/suggest {
     ingredients: [{ name, category }],  // from frontend state
     filters: { cookingTime, skillLevel, cuisineType, servings },
     userPreferences: { dietaryPreferences, cookingSkillLevel, dailyCalorieTarget }
   }
8. Recipe Service receives request (userId from JWT context)
9. Recipe Service reads IngredientsTable directly:
   DynamoDB Query: PK=USER#{userId}, SK begins_with ING#
   [Note: Q25-B — Lambda reads authoritative server-side ingredient list]
   [Note: Frontend-passed list used for prompt context; server-side read ensures consistency]
10. Recipe Service builds Bedrock prompt:
    - Ingredients available (from DynamoDB read)
    - Dietary constraints
    - Skill level cap
    - Cooking time target
    - Cuisine preference
    - Calorie target per serving
    - Servings count
    - Request: generate exactly 3 recipes
    - Prompt designed for < 30s completion (BR-RA-04)
11. Recipe Service → Bedrock (Claude 4.6 Sonnet): invoke with structured JSON output schema
12. Bedrock returns 3 recipes with: name, description, ingredients, steps, nutrition, prepTime, cookTime, difficulty, servings, dietaryTags, cuisineType
13. Recipe Service maps response to RecipeSuggestion[] format
14. Recipe Service computes missingIngredients for each recipe:
    - Compare recipe.ingredients against user's DynamoDB ingredient list
    - Items not found in inventory → missingIngredients[]
15. Return { suggestions: RecipeSuggestion[] }
16. Frontend stores result in PouchDB cache (RECIPE_CACHE#{userId})
17. Display RecipeSuggestionsView
```

### Load More Suggestions
```
1. User taps "Show more recipes"
2. Same flow as steps 4–17 above
3. New Bedrock call with same filters (no cache used for "load more")
4. Append new 3 suggestions to existing list
5. Update PouchDB cache with full combined list
```

---

## 4. Recipe Detail Flow (US-14)

```
1. User taps a recipe card (suggestion or saved)
2. Navigate to RecipeDetailPage
3. If suggestion (not yet saved):
   - Display full recipe from in-memory suggestion object
   - Show "Save Recipe" button
   - Missing ingredients highlighted in ingredient list
4. If saved recipe:
   - Fetch from PouchDB cache or GET /recipes/{recipeId}
   - Show "Notes" field (editable)
   - Show "Delete" option
5. User taps "Save Recipe":
   a. If online:
      - Check for duplicate name (GET /recipes/saved, filter by name)
      - If duplicate found: show warning toast "A recipe named '[name]' is already saved. Save anyway?"
      - User confirms → POST /recipes/saved { ...recipe, notes: null }
      - Recipe Service → DynamoDB: PutItem (RecipesTable)
      - Update PouchDB saved recipes cache
   b. If offline: show "Saving requires internet connection"
6. User edits notes on saved recipe:
   a. If online: PUT /recipes/saved/{recipeId} { notes }
   b. If offline: queue mutation in PouchDB, sync on reconnect
```

---

## 5. Recipe Filter Flow (US-15)

```
1. User opens filter panel on RecipeAdvisorPage (before or after suggestions shown)
2. Filter options displayed:
   - Cooking time: <15 min / 15–30 min / 30–60 min / >60 min (or no filter)
   - Skill level: beginner / intermediate / advanced (capped at user's profile level)
   - Cuisine type: Italian / Asian / Mediterranean / Mexican / Indian / Other (or no filter)
   - Servings: number input (1–10, default 2)
   - Dietary: profile preferences shown as read-only tags + optional one-time additions
3. User applies filters
4. If suggestions already shown:
   - Invalidate current cache entry for this filter combination
   - Trigger new suggestion request (step 3 of Recipe Suggestion Flow)
5. If no suggestions yet:
   - Filters stored in local state, applied on next "Get Recipes" tap
6. If Bedrock returns 0 matching recipes:
   - Show: "No recipes found with these filters. Try relaxing your filters."
   - Show "Clear Filters" button
```

---

## 6. Saved Recipes Flow

```
1. User navigates to SavedRecipesPage (top-level nav item)
2. If online:
   a. GET /recipes/saved
   b. Recipe Service → DynamoDB: Query PK=USER#{userId}, SK begins_with RECIPE#
   c. Return { recipes: SavedRecipe[] }
   d. Update PouchDB saved recipes cache
3. If offline:
   a. Read from PouchDB SAVED_RECIPES#{userId}
   b. Show stale indicator if cache > 24h old
4. Display recipe list (name, cuisine, total time, difficulty)
5. User taps recipe → RecipeDetailPage (saved recipe view)
6. User deletes recipe:
   a. If online: DELETE /recipes/saved/{recipeId}
   b. If offline: show "Deleting requires internet connection"
```

---

## 7. Offline Sync Integration (inherits Foundation SyncProvider)

```
Recipe Advisor mutations that go through the sync queue:
- Ingredient add (POST /ingredients)
- Ingredient update (PUT /ingredients/{id})
- Ingredient delete (DELETE /ingredients/{id})
- Ingredient bulk add (POST /ingredients/bulk)
- Recipe notes update (PUT /recipes/saved/{id})

After sync completes (SyncProvider callback):
- Re-fetch ingredient list from server
- Invalidate recipe suggestion cache (ingredients may have changed)
- Update PouchDB caches with fresh server data
```

---

## 8. Category Lookup Logic

```
Frontend utility: categorizIngredient(name: string): IngredientCategory

Algorithm:
1. Normalize name: lowercase, trim whitespace
2. Check against keyword map (partial match):
   - protein keywords: ["chicken", "beef", "pork", "lamb", "fish", "salmon", "tuna", "shrimp", "egg", "tofu", "turkey", "bacon"]
   - vegetable keywords: ["carrot", "spinach", "broccoli", "tomato", "onion", "garlic", "pepper", "lettuce", "cucumber", "zucchini", "potato", "mushroom"]
   - fruit keywords: ["apple", "banana", "orange", "strawberry", "lemon", "lime", "mango", "grape", "peach", "pear", "berry"]
   - dairy keywords: ["milk", "cheese", "yogurt", "butter", "cream", "cheddar", "mozzarella"]
   - grain keywords: ["rice", "pasta", "bread", "flour", "oat", "quinoa", "barley", "wheat", "noodle", "tortilla"]
   - spice keywords: ["salt", "pepper", "cumin", "paprika", "oregano", "basil", "cinnamon", "turmeric", "ginger", "chili"]
3. If match found: return matched category
4. If no match: return "other"
5. This is a suggestion only — user can always override
```
