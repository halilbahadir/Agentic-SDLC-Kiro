# Business Rules — Recipe Advisor Unit

## BR-RA-01: Ingredient Validation

| Rule | Constraint | Error Code |
|------|-----------|------------|
| Name required | Ingredient name must not be empty | INGREDIENT_NAME_REQUIRED |
| Name length | 1–100 characters | INGREDIENT_NAME_TOO_LONG |
| Name characters | Alphanumeric, spaces, hyphens only | INGREDIENT_NAME_INVALID |
| Category required | Must be one of the allowed IngredientCategory values | INGREDIENT_CATEGORY_INVALID |
| Quantity range | If provided, must be > 0 and ≤ 99999 | INGREDIENT_QUANTITY_INVALID |
| Unit required with quantity | If quantity is provided, unit must also be provided | INGREDIENT_UNIT_REQUIRED |
| Unit allowed values | Must be one of the allowed QuantityUnit values | INGREDIENT_UNIT_INVALID |
| Inventory limit | Maximum 100 ingredients per user | INGREDIENT_LIMIT_EXCEEDED |
| Duplicate check | Ingredient with same name (case-insensitive) already exists | INGREDIENT_DUPLICATE |

---

## BR-RA-02: Ingredient Category Auto-Suggestion

| Rule | Description |
|------|-------------|
| Auto-suggest on add | When user types an ingredient name, system suggests a category based on a lookup table |
| Lookup table | Common ingredients mapped to categories (e.g., "chicken" → "protein", "spinach" → "vegetable") |
| Fallback | If name not in lookup table, suggest "other" |
| User override | User can always change the suggested category before saving |
| Fridge scan | AI-returned category is used as the suggestion; user can override during confirmation |

### Category Lookup Examples
| Ingredient Pattern | Suggested Category |
|-------------------|-------------------|
| chicken, beef, pork, fish, salmon, tuna, egg, tofu | protein |
| carrot, spinach, broccoli, tomato, onion, garlic, pepper | vegetable |
| apple, banana, orange, strawberry, lemon | fruit |
| milk, cheese, yogurt, butter, cream | dairy |
| rice, pasta, bread, flour, oats, quinoa | grain |
| salt, pepper, cumin, paprika, oregano, basil, cinnamon | spice |

---

## BR-RA-03: Fridge Scan Rules

| Rule | Constraint | Error Code |
|------|-----------|------------|
| Image required | Must upload an image before scanning | IMAGE_REQUIRED |
| Image size | Maximum 10MB (inherited from shared ImageUpload component) | IMAGE_TOO_LARGE |
| Image format | JPEG, PNG, WEBP only | IMAGE_FORMAT_INVALID |
| Confidence filter | Items with confidence < 0.3 are excluded from results | — (silently filtered) |
| Minimum results | If 0 items pass confidence threshold, show message: "No ingredients could be identified. Try a clearer photo or add items manually." | FRIDGE_SCAN_NO_RESULTS |
| Merge behavior | Scan results are always merged with existing inventory (never replace) | — |
| Duplicate handling | If scanned ingredient name matches existing (case-insensitive), skip it silently | — |
| Confirmation required | User must confirm/deselect items before they are saved — no auto-save | — |
| Online required | Fridge scan requires internet connection (Bedrock call) | OFFLINE_FEATURE_UNAVAILABLE |

### Fridge Scan Confirmation Rules
| Rule | Description |
|------|-------------|
| All items pre-selected | All items above confidence threshold are pre-checked |
| User can deselect | User unchecks items they don't want to add |
| User can edit | User can change name, category, quantity before saving |
| Bulk save | Single "Add to Inventory" button saves all checked items |
| Category editable | Each item shows auto-suggested category with override option |

---

## BR-RA-04: Recipe Suggestion Rules

| Rule | Constraint | Error Code |
|------|-----------|------------|
| Online required | Recipe generation requires internet connection | OFFLINE_FEATURE_UNAVAILABLE |
| Ingredient list sent | Frontend passes current ingredient list in request body | — |
| Minimum ingredients | No minimum — generate even with 1–2 ingredients, mark missing extras clearly | — |
| AI inputs | Ingredient list + dietary preferences + skill level + calorie target + cooking time filter + servings | — |
| Initial count | Always generate 3 suggestions per request | — |
| Load more | "Show more" generates 3 additional suggestions (new Bedrock call) | — |
| Prompt complexity | Prompt must be designed to complete within 30 seconds (Q23-D constraint) | — |
| Missing ingredients | Each suggestion must clearly list which ingredients are NOT in user's inventory | — |
| Dietary compliance | All suggestions must respect user's dietary preferences from profile | — |
| Skill compliance | All suggestions must be at or below user's cooking skill level | — |

### Recipe Suggestion Cache Rules
| Rule | Description |
|------|-------------|
| Cache TTL | 14 days from generation time |
| Cache key | Combination of: sorted ingredient names + active filters |
| Cache hit | If ingredient list and filters unchanged AND cache not expired → return cached results |
| Cache invalidation | Any ingredient add/remove/update invalidates the cache immediately |
| Cache storage | PouchDB `RECIPE_CACHE#{userId}` document |
| Cache miss | Call Bedrock, store result in cache |

---

## BR-RA-05: Recipe Filter Rules

| Filter | Options | Behavior |
|--------|---------|---------|
| Cooking time | under_15 / 15_to_30 / 30_to_60 / over_60 (minutes, total = prep + cook) | Sent to Bedrock as constraint |
| Skill level | beginner / intermediate / advanced | Show recipes AT OR BELOW user's profile skill level; user can temporarily lower (not raise) |
| Dietary | Profile preferences always applied; user can add one-time overrides for this session | Sent to Bedrock as constraint |
| Cuisine type | Free text or from common list (Italian, Asian, Mediterranean, Mexican, Indian, Other) | Sent to Bedrock as constraint |
| Servings | Number (1–10) | Sent to Bedrock; recipe nutrition scaled accordingly |

### Filter Validation Rules
| Rule | Description |
|------|-------------|
| All filters optional | No filter is required; defaults: no time filter, profile skill level, profile dietary prefs, no cuisine, 2 servings |
| Skill level cap | User cannot request recipes above their own skill level |
| No results | If Bedrock returns 0 matching recipes: show "No recipes found with these filters. Try relaxing your filters." |

---

## BR-RA-06: Recipe Save Rules

| Rule | Constraint | Error Code |
|------|-----------|------------|
| Online required for initial save | Saving requires internet (writes to DynamoDB) | OFFLINE_FEATURE_UNAVAILABLE |
| Duplicate name warning | If a saved recipe with the same name (case-insensitive) already exists, warn user but allow save | — (warning, not error) |
| Notes field | Optional, max 1000 characters | RECIPE_NOTES_TOO_LONG |
| Notes editable offline | User can edit notes on a saved recipe while offline — queued for sync | — |
| Max saved recipes | No hard limit | — |
| Delete | User can delete any saved recipe; deletion is permanent | — |

---

## BR-RA-07: Offline Behavior Rules

| Feature | Offline Behavior |
|---------|-----------------|
| View saved recipes | ✅ Available — served from PouchDB cache |
| View ingredient list | ✅ Available — served from PouchDB cache |
| Edit ingredient list | ✅ Available — changes queued in PouchDB, synced on reconnect |
| Add ingredient manually | ✅ Available — queued for sync |
| Remove ingredient | ✅ Available — queued for sync |
| Fridge photo scan | ❌ Blocked — show "Fridge scan requires internet connection" |
| Generate recipe suggestions | ❌ Blocked — show "Recipe generation requires internet connection" |
| View cached suggestions | ✅ Available — if cache exists and not expired (14-day TTL) |
| Save recipe | ❌ Blocked — show "Saving requires internet connection" |
| Edit recipe notes | ✅ Available — queued for sync |

### Offline Sync for Ingredient Changes (inherits Foundation BR-06 pattern)
| Rule | Description |
|------|-------------|
| Queue on write | Ingredient mutations stored in PouchDB with `synced: false` |
| Sync on reconnect | FIFO order, same retry pattern as Foundation (3 retries, exponential backoff) |
| Conflict resolution | Server-wins (same as Foundation) |
| Cache invalidation | After sync completes, recipe suggestion cache is invalidated (ingredients changed) |

---

## BR-RA-08: Recipe Detail Rules

| Rule | Description |
|------|-------------|
| Nutrition source | AI generates full recipe including nutrition in one Bedrock call (Q11-A) |
| Nutrition display | Per-serving values: calories, protein, carbs, fat, fiber |
| Total time display | prepTime + cookTime shown as "Total: X min" |
| Missing ingredients highlight | Ingredients not in user's inventory are visually highlighted in the recipe detail |
| Save from detail | User can save recipe directly from the detail view |
| Notes on saved | Notes field only appears on saved recipes (not on unsaved suggestions) |
