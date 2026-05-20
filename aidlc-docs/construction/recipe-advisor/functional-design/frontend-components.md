# Frontend Components — Recipe Advisor Unit

## Lazy Bundle Entry Points

Per Foundation NFR design (code splitting), Recipe Advisor contributes two lazy-loaded bundles:

```typescript
// In App.tsx (Foundation)
const IngredientsModule = React.lazy(() => import('./modules/ingredient-management'));
const RecipesModule = React.lazy(() => import('./modules/recipe-advisor'));
```

- `ingredients.js` → entry: `src/frontend/modules/ingredient-management/index.ts`
- `recipes.js` → entry: `src/frontend/modules/recipe-advisor/index.ts`

---

## Component Hierarchy

```
App (Foundation)
└── ProtectedRoutes
    └── AppLayout
        ├── NavigationBar (Foundation — "Ingredients" + "Recipes" items)
        │
        ├── /ingredients  → IngredientListPage (lazy: ingredients.js)
        │     ├── IngredientCategoryGroup (×N categories)
        │     │   └── IngredientItem (×N items)
        │     ├── AddIngredientModal
        │     │   └── CategorySelector
        │     ├── EditIngredientModal
        │     │   └── CategorySelector
        │     └── FridgeScanPage (sub-route or modal)
        │         └── FridgeScanResultsView
        │             └── FridgeScanIngredientRow (×N items)
        │
        ├── /recipes  → RecipeAdvisorPage (lazy: recipes.js)
        │     ├── RecipeFilterPanel
        │     ├── RecipeSuggestionsView
        │     │   └── RecipeCard (×N)
        │     └── RecipeDetailPage (sub-route)
        │         └── RecipeIngredientRow (×N)
        │
        └── /recipes/saved  → SavedRecipesPage (lazy: recipes.js)
              └── SavedRecipeCard (×N)
```

---

## Ingredient Management Module

### IngredientListPage
**Route**: `/ingredients`  
**Lazy bundle**: `ingredients.js`

| State | Type | Description |
|-------|------|-------------|
| ingredients | Ingredient[] | Full inventory from useIngredients hook |
| isLoading | boolean | Fetch loading state |
| error | string \| null | Fetch error |
| showAddModal | boolean | Controls AddIngredientModal visibility |
| editingIngredient | Ingredient \| null | Ingredient being edited (controls EditIngredientModal) |
| isOnline | boolean | From SyncProvider context |

**User Interactions:**
- View ingredient list grouped by category
- Tap FAB (+) → open AddIngredientModal
- Tap ingredient row → open EditIngredientModal
- Swipe left on ingredient → delete confirmation → remove
- Tap "Scan Fridge" button (top of page) → navigate to FridgeScanPage
- Pull-to-refresh → re-fetch from server (online only)

**API Integration:** `GET /ingredients`, `DELETE /ingredients/{id}`

**Offline behavior:** Shows cached ingredient list from PouchDB; FAB and swipe-delete still work (queued); "Scan Fridge" button disabled with tooltip "Requires internet"

**data-testid conventions:**
```
ingredient-list-page
ingredient-list-scan-fridge-button
ingredient-list-add-fab
ingredient-list-pull-refresh
ingredient-item-{ingredientId}
ingredient-item-{ingredientId}-delete-button
ingredient-category-group-{category}
```

---

### IngredientItem
**Purpose**: Single ingredient row in the list

| Prop | Type | Description |
|------|------|-------------|
| ingredient | Ingredient | The ingredient data |
| onEdit | (ingredient: Ingredient) => void | Open edit modal |
| onDelete | (ingredientId: string) => void | Trigger delete |

**Displays:** Name, quantity + unit (if set), category badge, sync-pending indicator (if offline queued)

**data-testid conventions:**
```
ingredient-item-name-{ingredientId}
ingredient-item-category-badge-{ingredientId}
ingredient-item-quantity-{ingredientId}
ingredient-item-sync-pending-{ingredientId}
```

---

### AddIngredientModal
**Purpose**: Form to add a new ingredient manually

| State | Type | Description |
|-------|------|-------------|
| name | string | Ingredient name input |
| suggestedCategory | IngredientCategory | Auto-suggested from lookup |
| selectedCategory | IngredientCategory | User-selected (defaults to suggested) |
| quantity | string | Numeric quantity input |
| unit | QuantityUnit \| null | Selected unit |
| errors | Record<string, string> | Field-level validation errors |
| isSubmitting | boolean | Submission loading state |

**User Interactions:**
- Type name → category auto-suggested
- Override category via CategorySelector
- Enter quantity + select unit (both optional)
- Submit → validate → call useIngredients().add()
- Cancel → close modal

**Form Validation (client-side, per BR-RA-01):**
- Name: required, 1–100 chars
- Quantity: if entered, must be > 0
- Unit: required if quantity entered

**API Integration:** via `useIngredients` hook → `POST /ingredients`

**data-testid conventions:**
```
add-ingredient-modal
add-ingredient-name-input
add-ingredient-category-selector
add-ingredient-quantity-input
add-ingredient-unit-selector
add-ingredient-submit-button
add-ingredient-cancel-button
```

---

### EditIngredientModal
**Purpose**: Edit an existing ingredient's quantity, unit, or category

| Prop | Type | Description |
|------|------|-------------|
| ingredient | Ingredient | Current ingredient values |
| onSave | (updates: Partial<Ingredient>) => void | Save callback |
| onClose | () => void | Close modal |

**State:** Same as AddIngredientModal but pre-populated with existing values.

**data-testid conventions:**
```
edit-ingredient-modal
edit-ingredient-name-display
edit-ingredient-category-selector
edit-ingredient-quantity-input
edit-ingredient-unit-selector
edit-ingredient-save-button
edit-ingredient-cancel-button
```

---

### CategorySelector
**Purpose**: Reusable dropdown/chip selector for IngredientCategory

| Prop | Type | Description |
|------|------|-------------|
| value | IngredientCategory | Currently selected category |
| onChange | (category: IngredientCategory) => void | Selection callback |
| suggested | IngredientCategory \| null | Highlighted as "suggested" |

**Displays:** 7 category options as chips or dropdown. Suggested category shown with "suggested" label.

**data-testid conventions:**
```
category-selector
category-selector-option-{category}
```

---

### FridgeScanPage
**Route**: `/ingredients/scan` (or modal overlay)  
**Purpose**: Fridge photo capture and AI ingredient identification

| State | Type | Description |
|-------|------|-------------|
| scanPhase | "capture" \| "scanning" \| "results" \| "error" | Current phase |
| imageFile | File \| null | Selected image |
| scanResults | FridgeScanResult[] | AI-returned items (filtered by confidence) |
| isSubmitting | boolean | Saving confirmed items |

**User Interactions:**
- Phase "capture": ImageUpload component (camera/gallery) → validate → start scan
- Phase "scanning": LoadingSpinner + "Identifying ingredients..." message
- Phase "results": FridgeScanResultsView
- Phase "error": ErrorMessage + retry option

**API Integration:** `POST /images/upload-url` → S3 upload → `POST /ai/analyze/fridge`

**data-testid conventions:**
```
fridge-scan-page
fridge-scan-image-upload
fridge-scan-loading
fridge-scan-error-message
fridge-scan-retry-button
```

---

### FridgeScanResultsView
**Purpose**: Confirmation screen after AI identifies fridge ingredients

| Prop | Type | Description |
|------|------|-------------|
| results | FridgeScanResult[] | AI-identified items (confidence ≥ 0.3) |
| onConfirm | (selected: FridgeScanResult[]) => void | Save selected items |
| onCancel | () => void | Discard and go back |

| State | Type | Description |
|-------|------|-------------|
| selectedItems | Set<string> | IDs of checked items |
| editedItems | Map<string, Partial<FridgeScanResult>> | User edits to name/category |

**User Interactions:**
- All items pre-selected (checkbox)
- Uncheck to deselect
- Tap item name to edit inline
- Tap category badge to override via CategorySelector
- "Add X items to inventory" button → calls onConfirm with selected items

**data-testid conventions:**
```
fridge-scan-results-view
fridge-scan-results-confirm-button
fridge-scan-results-cancel-button
fridge-scan-ingredient-row-{index}
fridge-scan-ingredient-checkbox-{index}
fridge-scan-ingredient-name-input-{index}
fridge-scan-ingredient-confidence-badge-{index}
```

---

## Recipe Advisor Module

### RecipeAdvisorPage
**Route**: `/recipes`  
**Lazy bundle**: `recipes.js`

| State | Type | Description |
|-------|------|-------------|
| suggestions | RecipeSuggestion[] | Current suggestion list |
| filters | RecipeFilters | Active filter state |
| isGenerating | boolean | Bedrock call in progress |
| isLoadingMore | boolean | "Show more" in progress |
| error | string \| null | Generation error |
| showFilterPanel | boolean | Filter panel visibility |
| ingredientCount | number | From useIngredients (summary display) |
| isOnline | boolean | From SyncProvider |

**Layout (single-page scroll, Q19-A):**
```
┌─────────────────────────────────┐
│ 🥦 X ingredients in your fridge │  ← ingredient summary
│ [Filter] [Get Recipes]          │  ← action bar
├─────────────────────────────────┤
│ RecipeFilterPanel (collapsible) │
├─────────────────────────────────┤
│ RecipeCard × 3                  │  ← suggestions
│ [Show more recipes]             │  ← load more button
└─────────────────────────────────┘
```

**User Interactions:**
- Tap "Get Recipes" → trigger suggestion flow
- Tap "Filter" → expand/collapse RecipeFilterPanel
- Tap recipe card → navigate to RecipeDetailPage
- Tap "Show more recipes" → load 3 more
- If offline: "Get Recipes" disabled with tooltip

**data-testid conventions:**
```
recipe-advisor-page
recipe-advisor-ingredient-summary
recipe-advisor-get-recipes-button
recipe-advisor-filter-toggle-button
recipe-advisor-loading
recipe-advisor-error-message
recipe-advisor-show-more-button
```

---

### RecipeFilterPanel
**Purpose**: Collapsible filter controls for recipe generation

| Prop | Type | Description |
|------|------|-------------|
| filters | RecipeFilters | Current filter values |
| userSkillLevel | DifficultyLevel | From profile (caps skill filter) |
| userDietaryPrefs | string[] | From profile (shown as read-only) |
| onChange | (filters: RecipeFilters) => void | Filter change callback |

**Controls:**
- Cooking time: 4-option chip selector (under_15 / 15_to_30 / 30_to_60 / over_60 / any)
- Skill level: 3-option chip selector (capped at user's profile level)
- Cuisine type: dropdown (Italian / Asian / Mediterranean / Mexican / Indian / Other / any)
- Servings: number stepper (1–10, default 2)
- Dietary: profile prefs shown as read-only tags + "Add override" chip

**data-testid conventions:**
```
recipe-filter-panel
recipe-filter-cooking-time-{option}
recipe-filter-skill-level-{level}
recipe-filter-cuisine-selector
recipe-filter-servings-stepper
recipe-filter-servings-increment
recipe-filter-servings-decrement
recipe-filter-dietary-tag-{pref}
recipe-filter-clear-button
```

---

### RecipeCard
**Purpose**: Summary card for a recipe suggestion

| Prop | Type | Description |
|------|------|-------------|
| recipe | RecipeSuggestion | Recipe data |
| onTap | () => void | Navigate to detail |

**Displays:**
- Recipe name
- Cuisine type badge (if set)
- Total time (prep + cook)
- Difficulty badge
- Calories per serving
- Missing ingredients count (e.g., "2 extra ingredients needed") — highlighted in amber if > 0
- Dietary tags (up to 3, then "+N more")

**data-testid conventions:**
```
recipe-card-{suggestionId}
recipe-card-name-{suggestionId}
recipe-card-time-{suggestionId}
recipe-card-difficulty-badge-{suggestionId}
recipe-card-missing-ingredients-{suggestionId}
recipe-card-save-button-{suggestionId}
```

---

### RecipeDetailPage
**Route**: `/recipes/{suggestionId}` (suggestion) or `/recipes/saved/{recipeId}` (saved)

| State | Type | Description |
|-------|------|-------------|
| recipe | RecipeSuggestion \| SavedRecipe | Recipe being viewed |
| isSaving | boolean | Save operation in progress |
| notes | string | Notes field value (saved recipes only) |
| isEditingNotes | boolean | Notes edit mode |
| showDuplicateWarning | boolean | Duplicate name warning |

**Layout:**
```
┌─────────────────────────────────┐
│ ← Back    [Save Recipe]         │
│ Recipe Name                     │
│ Cuisine • X min • Difficulty    │
│ Calories: X kcal/serving        │
├─────────────────────────────────┤
│ Ingredients                     │
│  ✅ Chicken Breast (in fridge)  │
│  ⚠️  Lemon (not in fridge)      │
├─────────────────────────────────┤
│ Instructions                    │
│  1. ...                         │
│  2. ...                         │
├─────────────────────────────────┤
│ Nutrition (per serving)         │
│  Calories / Protein / Carbs / Fat│
├─────────────────────────────────┤
│ Notes (saved recipes only)      │
│  [editable text area]           │
└─────────────────────────────────┘
```

**User Interactions:**
- Tap "Save Recipe" → save flow (BR-RA-06)
- Tap ingredient row → no action (display only)
- Tap "Edit Notes" → inline notes editing
- Tap "Delete Recipe" (saved only) → confirmation → delete

**data-testid conventions:**
```
recipe-detail-page
recipe-detail-name
recipe-detail-save-button
recipe-detail-delete-button
recipe-detail-ingredient-row-{index}
recipe-detail-ingredient-available-{index}
recipe-detail-ingredient-missing-{index}
recipe-detail-step-{index}
recipe-detail-nutrition-calories
recipe-detail-nutrition-protein
recipe-detail-nutrition-carbs
recipe-detail-nutrition-fat
recipe-detail-notes-field
recipe-detail-notes-edit-button
recipe-detail-notes-save-button
recipe-detail-duplicate-warning
recipe-detail-duplicate-confirm-button
```

---

### SavedRecipesPage
**Route**: `/recipes/saved`  
**Lazy bundle**: `recipes.js` (same bundle as RecipeAdvisorPage)

| State | Type | Description |
|-------|------|-------------|
| savedRecipes | SavedRecipe[] | From useSavedRecipes hook |
| isLoading | boolean | Fetch state |
| error | string \| null | Fetch error |
| isOnline | boolean | From SyncProvider |

**Layout:** List of SavedRecipeCard components, grouped by nothing (chronological, newest first).

**Offline behavior:** Shows PouchDB cached saved recipes; stale indicator if cache > 24h.

**data-testid conventions:**
```
saved-recipes-page
saved-recipes-list
saved-recipe-card-{recipeId}
saved-recipe-card-name-{recipeId}
saved-recipe-card-time-{recipeId}
saved-recipes-empty-state
saved-recipes-stale-indicator
```

---

## Custom Hooks

### useIngredients()
**Purpose**: Ingredient inventory CRUD with offline support

```typescript
interface UseIngredientsReturn {
  ingredients: Ingredient[];
  isLoading: boolean;
  error: string | null;
  add: (data: CreateIngredientInput) => Promise<void>;
  update: (id: string, updates: Partial<Ingredient>) => Promise<void>;
  remove: (id: string) => Promise<void>;
  bulkAdd: (items: CreateIngredientInput[]) => Promise<void>;
  refetch: () => Promise<void>;
}
```

**Offline behavior:** Reads from PouchDB when offline; writes go to PouchDB queue via SyncProvider.queueMutation().

---

### useRecipeSuggestions()
**Purpose**: Recipe generation with cache management

```typescript
interface UseRecipeSuggestionsReturn {
  suggestions: RecipeSuggestion[];
  isGenerating: boolean;
  isLoadingMore: boolean;
  error: string | null;
  generate: (filters: RecipeFilters) => Promise<void>;
  loadMore: (filters: RecipeFilters) => Promise<void>;
  clearSuggestions: () => void;
}
```

**Cache logic:** Checks PouchDB RECIPE_CACHE before calling API. Invalidates cache when ingredients change.

---

### useSavedRecipes()
**Purpose**: Saved recipe CRUD

```typescript
interface UseSavedRecipesReturn {
  savedRecipes: SavedRecipe[];
  isLoading: boolean;
  error: string | null;
  save: (recipe: RecipeSuggestion, notes?: string) => Promise<void>;
  updateNotes: (recipeId: string, notes: string) => Promise<void>;
  remove: (recipeId: string) => Promise<void>;
  refetch: () => Promise<void>;
}
```

---

## Context Integration

Recipe Advisor components consume these Foundation contexts:

| Context | Used By | What it provides |
|---------|---------|-----------------|
| AuthProvider | All hooks | userId for API calls |
| SyncProvider | useIngredients | queueMutation(), isOnline, triggerSync callback |
| ThemeProvider | All components | theme for Tailwind dark mode classes |
