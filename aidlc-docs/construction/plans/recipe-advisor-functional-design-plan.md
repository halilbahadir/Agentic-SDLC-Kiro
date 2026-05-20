# Recipe Advisor Unit — Functional Design Plan

---

## 👥 Unit Team

| Member | Role | Responsibility |
|--------|------|---------------|
| awsHalil ⭐ | Developer | Story development (frontend) — Ingredient Mgmt Module, Recipe Advisor Module, AWS Bedrock integration (Lead) |
| halilbahadir | Tech Lead | Story development (backend) — Recipe Service, Ingredient Service |

> 📣 **awsHalil**: This is a collaborative session. Please invite halilbahadir to review these questions together — backend design decisions (service logic, data access patterns, Bedrock prompt strategy) benefit from Tech Lead input. Frontend decisions are yours to drive.

---

## 📐 Inherited Design Principles (MULTIDEV-07)

The following principles were established during Application Design and **must be followed** unless a deviation is formally proposed.

| Principle | Constraint |
|-----------|-----------|
| API Style | REST with resource-based URLs — all backend endpoints follow `/resource/:id` pattern |
| Auth | Cognito JWT validated at API Gateway level — Lambda receives pre-validated `userId` in event context |
| State Management | React Context + useReducer — no external state libraries (Redux, Zustand, etc.) |
| Error Format | `{ error: { code, message, details } }` — consistent across all services |
| Logging | Structured JSON: `{ level, service, method, userId, timestamp, duration }` — ERROR level only |
| Data Isolation | All DynamoDB access scoped by `USER#{userId}` partition key |
| Offline-First | Frontend assumes offline capability; AI features (Bedrock calls) are online-only |
| HTTP Client | axios with custom hooks — no direct fetch() in components |
| Shared Types | `src/shared/types.ts` — `Ingredient`, `Recipe`, `NutritionData` already defined there |
| Service Independence | No Lambda-to-Lambda calls — Recipe Service reads DynamoDB **directly** (DynamoDB SDK), not via Ingredient Service HTTP API |
| AI Model | **AWS Bedrock Claude 4.6 Sonnet** — already decided in Foundation tech-stack. Not open for re-selection. |
| Lambda Config | RecipesFunction and IngredientsFunction: **512MB / 60s timeout** (AI calls) — hard ceiling for Bedrock recipe generation |
| Code Splitting | `ingredients.js` and `recipes.js` are **separate lazy-loaded bundles** — frontend-components.md must define the route-level lazy entry point for each |
| PouchDB Cache | Shared 30-day / 50MB cache budget across all units — Recipe Advisor must define what it stores (saved recipes, ingredient list) and respect the 24h TTL stale policy |
| data-testid | All interactive UI elements must have `data-testid` attributes using pattern `{component}-{element-role}` (e.g., `recipe-card-save-button`) — define naming conventions in frontend-components.md |

## 🔗 Contracts You Consume (Cross-Unit)

| Provider | Endpoint | Contract |
|----------|----------|---------|
| AI Analysis Service (Calorie Tracking unit) | `POST /ai/analyze/fridge` | Request: `{ imageKey: string, userId: string }` → Response: `{ ingredients[]: { name, category, confidence } }` |

**Note**: This endpoint is owned by halilbahadir's Calorie Tracking unit. During Recipe Advisor development, mock this endpoint locally. Integration happens when Calorie Tracking delivers the endpoint.

## 🔗 Contracts You Provide

| Consumer | What you provide |
|----------|-----------------|
| None — Recipe Advisor is a leaf unit (no other unit depends on it) | — |

---

## Unit Context

| Attribute | Value |
|-----------|-------|
| **Unit** | Recipe Advisor |
| **Branch** | unit/recipe-advisor |
| **Stories** | US-11, US-12, US-13, US-14, US-15 (all MVP) |
| **Depends On** | Foundation (auth, infra, shared types, shared UI, offline sync) |
| **Depends On (partial)** | Calorie Tracking — `POST /ai/analyze/fridge` endpoint (US-11 only) |
| **Provides To** | Nothing — leaf unit |

---

## Functional Design Steps

- [x] Step 1: Analyze unit context and inherited principles
- [x] Step 2: Collect and analyze answers to questions below (25 questions across 9 sections)
- [x] Step 3: Generate `domain-entities.md`
- [x] Step 4: Generate `business-rules.md`
- [x] Step 5: Generate `business-logic-model.md`
- [x] Step 6: Generate `frontend-components.md`
- [x] Step 7: Present completion message and await approval

---

## Questions

Answer each question by filling in the `[Answer]:` tag below it.

---

### Section 1: Ingredient Domain Model

**Q1** — The shared `Ingredient` type defines: `{ id, name, category, quantity? }`. For the ingredient inventory, do you need to track additional fields per ingredient?

```
A) No — name, category, quantity is sufficient
B) Yes — also track unit of measurement (e.g., "500g", "2 cups", "3 pieces")
C) Yes — also track expiry date / "use by" date
D) Yes — both unit of measurement AND expiry date
E) Other (describe)
```
[Answer]: B

---

**Q2** — When a user manually adds an ingredient, how is the category assigned?

```
A) Auto-assigned by the system based on ingredient name (AI or lookup table) — user cannot change it
B) Auto-suggested by the system, but user can override the category
C) User always selects the category manually from a dropdown
D) No categorization — ingredients are just a flat list
```
[Answer]: B

---

**Q3** — For ingredient quantity tracking, what level of detail is needed?

```
A) Free-text only (e.g., user types "half a bag", "some", "3") — no structured quantity
B) Structured: number + unit (e.g., 500 grams, 2 cups) — validated input
C) Optional structured: user can enter structured or leave blank
D) No quantity tracking — just presence/absence of ingredient
```
[Answer]: B

---

### Section 2: Fridge Scan Flow (US-11)

**Q4** — After the AI returns identified ingredients from the fridge photo, what is the confirmation UX?

```
A) Show all identified items as a checklist — user confirms the whole list at once (bulk confirm)
B) Show items one by one — user confirms or rejects each individually
C) Show all items pre-selected — user can deselect/edit individual items before saving
D) Auto-save all items above a confidence threshold, show low-confidence items for review
```
[Answer]: C

---

**Q5** — What confidence threshold should be used to decide which AI-identified ingredients are shown vs. filtered out?

```
A) Show all items regardless of confidence (user decides what to keep)
B) Hide items below 0.3 confidence, show the rest
C) Hide items below 0.5 confidence, show the rest
D) Hide items below 0.7 confidence, show the rest
E) Let the user set their own threshold in settings
```
[Answer]: B

---

**Q6** — When the fridge scan returns ingredients, should they be **merged** with the existing ingredient list or **replace** it?

```
A) Always merge — add new items, keep existing ones untouched
B) Always replace — clear the current list and use scan results
C) Ask the user each time (merge or replace prompt)
D) Merge, but flag duplicates so user can decide
```
[Answer]: A

---

### Section 3: Recipe Suggestion Logic (US-13)

**Q7** — When generating recipe suggestions, what inputs does the AI receive? Select all that apply:

```
A) User's current ingredient inventory (always included)
B) User's dietary preferences from profile (always included)
C) User's cooking skill level from profile (always included)
D) User's daily calorie target from profile (always included)
E) User-specified cooking time preference (set at request time)
F) User-specified number of servings (set at request time)
```
[Answer]: (list all that apply, e.g. A, B, C) A, B, C, D, E, F

---

**Q8** — How many recipe suggestions should be generated per request, and can the user request more?

```
A) Always exactly 3 — no option to load more
B) Always exactly 5 — no option to load more
C) 3 initially, with a "Show more" button that generates 3 more
D) 5 initially, with a "Show more" button that generates 5 more
E) User selects how many (3, 5, or 10) before generating
```
[Answer]: C

---

**Q9** — What happens when the user has very few ingredients (e.g., only 1-2 items) and the AI cannot generate meaningful recipes?

```
A) Show an error message: "Not enough ingredients to suggest recipes. Add more items."
B) Generate recipes anyway, clearly marking which extra ingredients are needed to buy
C) Show a "minimal ingredients" warning but still attempt generation
D) Suggest the user add more ingredients first, with a shortcut to the ingredient list
```
[Answer]: B

---

**Q10** — Should recipe suggestions be cached? If the user taps "Suggest Recipes" again with the same ingredients, should it re-call Bedrock or return cached results?

```
A) Always re-call Bedrock — fresh suggestions every time (no caching)
B) Cache results for the current session only — re-call if ingredients change or app restarts
C) Cache results for 24 hours — only re-call if ingredients change significantly
D) Cache results indefinitely until ingredients change
```
[Answer]: C but 14 days cache

---

### Section 4: Recipe Detail & Save (US-14)

**Q11** — When a recipe is generated by AI, is the nutrition data calculated by the AI as part of the generation, or computed separately after generation?

```
A) AI generates the full recipe including nutrition in one Bedrock call
B) AI generates the recipe (ingredients + steps), then a separate nutrition estimation call is made
C) Nutrition is estimated client-side using the ingredient list (no extra API call)
D) Nutrition is not shown for AI-generated recipes — only for saved/confirmed recipes
```
[Answer]: A

---

**Q12** — When a user saves a recipe, what is saved to DynamoDB?

```
A) The full recipe object as returned by AI (ingredients, steps, nutrition, metadata)
B) A reference/ID only — recipe is re-generated on demand when viewed again
C) Full recipe object + a user-editable notes field
D) Full recipe object + user can edit ingredients/steps before saving
```
[Answer]: C

---

**Q13** — Can a user save the same recipe multiple times (e.g., they generate it twice and save both)?

```
A) Yes — each save creates a new entry, duplicates allowed
B) No — deduplicate by recipe name, show "already saved" if duplicate detected
C) No — deduplicate by content hash, silently skip if identical recipe already saved
D) Yes, but warn the user if a recipe with the same name already exists
```
[Answer]: D

---

### Section 5: Recipe Filtering (US-15)

**Q14** — Filters are applied to recipe suggestions. When are filters evaluated?

```
A) Pre-generation: filters are sent to Bedrock as constraints (AI only generates matching recipes)
B) Post-generation: AI generates recipes freely, then frontend filters the results
C) Hybrid: some filters (dietary, skill) go to Bedrock; time filter applied post-generation
D) User sets filters before requesting suggestions, and they are always sent to Bedrock
```
[Answer]: A

---

**Q15** — What filter options are available? (confirm or adjust)

```
Proposed filter set:
- Cooking time: <30 min / 30-60 min / >60 min
- Skill level: beginner / intermediate / advanced (show recipes AT OR BELOW user's level)
- Dietary: respect profile preferences (always on) + allow additional one-time overrides

A) Approve this filter set as-is
B) Add more time buckets (e.g., <15 min, 15-30 min, 30-60 min, >60 min)
C) Add a "max missing ingredients" filter (e.g., show only recipes needing 0-2 extra ingredients)
D) Add cuisine type filter (Italian, Asian, Mediterranean, etc.)
E) Multiple of the above (specify which)
```
[Answer]: A,B,D 

---

### Section 6: Offline Behavior

**Q16** — Which Recipe Advisor features should work offline?

```
A) View saved recipes only (no ingredient list, no suggestions)
B) View saved recipes + view/edit ingredient list (no AI features)
C) View saved recipes + view/edit ingredient list + view cached recipe suggestions
D) Everything except fridge photo scan and new recipe generation (all AI calls require online)
```
[Answer]: A

---

**Q17** — When the user adds/removes ingredients while offline, how is this handled?

```
A) Block ingredient changes while offline — show "requires internet connection"
B) Allow changes, queue them in PouchDB, sync when back online (same pattern as Foundation offline sync)
C) Allow changes locally only — changes are NOT synced to server (local-only ingredient list)
D) Allow changes, but warn the user that recipe suggestions will be stale until sync
```
[Answer]: B

---

### Section 7: Frontend Component Structure

**Q18** — For the Ingredient Management screen, what is the primary layout?

```
A) Single screen: ingredient list with FAB (floating action button) to add, swipe-to-delete
B) Two tabs on one screen: "My Ingredients" list tab + "Scan Fridge" tab
C) Separate screens: IngredientListPage (default) with navigation to FridgeScanPage
D) Ingredient list with a prominent "Scan Fridge" button at the top, scan opens a modal
```
[Answer]: A

---

**Q19** — For the Recipe Advisor screen, what is the primary navigation flow?

```
A) Single page: ingredient summary at top, "Get Recipes" button, results below (all on one scroll)
B) Two screens: RecipeListPage (shows suggestions) → RecipeDetailPage (full recipe)
C) Three screens: IngredientsConfirmPage → RecipeSuggestionsPage → RecipeDetailPage
D) Modal-based: suggestions appear in a bottom sheet over the ingredient list
```
[Answer]: A

---

**Q20** — Where do saved recipes live in the navigation?

```
A) Separate "Saved Recipes" tab/section in the Recipe Advisor screen
B) Accessible from the user profile page
C) Separate top-level navigation item (alongside Dashboard, Ingredients, Recipes)
D) Mixed into the recipe suggestions list (saved recipes shown first)
```
[Answer]: C

---

### Section 8: Error Handling & Edge Cases

**Q21** — What error code/message should be shown when Bedrock recipe generation fails (timeout or service error)?

```
A) Generic: "Something went wrong. Please try again."
B) Specific: "Recipe generation is temporarily unavailable. Please try again in a moment."
C) Specific + fallback: Show error + offer to show previously cached suggestions if available
D) Retry automatically once before showing error to user
```
[Answer]: B

---

**Q22** — What is the maximum number of ingredients a user can have in their inventory?

```
A) No limit
B) 50 ingredients max
C) 100 ingredients max
D) 200 ingredients max
```
[Answer]: C

---

### Section 9: Foundation Alignment (New — from cross-reference review)

**Q23** — Recipe generation via Bedrock has a hard 60-second Lambda timeout ceiling. What should happen if the Bedrock call takes longer than 55 seconds (approaching the limit)?

```
A) Let it timeout naturally — API Gateway returns 504, frontend shows generic error
B) Set an internal 50-second timeout on the Bedrock call, return a graceful error before Lambda times out
C) Split into two calls: first call starts generation and returns a job ID, second call polls for result
D) Reduce recipe complexity in the prompt to ensure generation completes within 30 seconds
```
[Answer]: D

---

**Q24** — Recipe Advisor shares the PouchDB cache budget (30 days / 50MB per user) with all other units. What should Recipe Advisor store locally in PouchDB?

```
A) Saved recipes only (ingredient list always fetched fresh from server)
B) Ingredient list only (saved recipes always fetched fresh from server)
C) Both saved recipes AND ingredient list
D) Neither — Recipe Advisor has no offline data (all features require internet)
```
[Answer]: C

---

**Q25** — The Recipe Service Lambda reads the IngredientsTable directly via DynamoDB SDK (not via HTTP call to Ingredient Service). When generating recipe suggestions, should the Recipe Service Lambda read the user's full ingredient list from DynamoDB as part of the suggestion request, or should the frontend pass the ingredient list in the request body?

```
A) Frontend passes the ingredient list in the POST /recipes/suggest request body
B) Recipe Service Lambda reads IngredientsTable directly using the userId from the JWT context
C) Either approach is fine — use whichever is simpler to implement
```
[Answer]: A

---
