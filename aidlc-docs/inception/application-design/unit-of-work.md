# Units of Work

## Decomposition: 3 Units with Foundation

### Code Organization (Flat Monorepo)
```
src/
  frontend/         (React app — all modules)
    components/
    modules/
      auth/
      meal-logging/
      nutrition-dashboard/
      ingredient-management/
      recipe-advisor/
      offline-sync/
    shared/
    App.tsx
  backend/          (Lambda handlers — all services)
    auth/
    meals/
    ai-analysis/
    recipes/
    ingredients/
    images/
    shared/         (middleware, utils, types)
  shared/           (cross-layer types and constants)
infra/              (CDK stacks)
```

---

## Unit 1: Foundation

| Attribute | Value |
|-----------|-------|
| **Name** | foundation |
| **Branch** | unit/foundation |
| **Stories** | US-01, US-02, US-03, US-17 (4 stories) |
| **Effort** | S + S + M + L = Medium-Large |

### Scope
- Project scaffolding (Vite + React + TypeScript setup)
- Shared types package (`src/shared/`)
- CDK infrastructure (API Gateway, DynamoDB tables, S3 buckets, Cognito User Pool, CloudFront)
- Auth Service (Lambda: `calorie-app-auth`)
- Auth frontend module (registration, login, password reset, profile setup)
- Offline Sync module (PouchDB setup, sync status, offline detection)
- Shared UI components (navigation, loading states, error boundaries, theme)
- CI/CD pipeline configuration (if applicable)

### Delivers
- Working authentication flow (register → login → profile setup)
- Deployed infrastructure (all AWS resources provisioned)
- Offline-capable shell with sync mechanism
- Shared types and utilities for other units to consume
- Base React app with routing and navigation

### Exit Criteria
- User can register, login, set up profile
- App works offline (shows cached data, queues mutations)
- All DynamoDB tables, S3 buckets, Cognito pool deployed
- API Gateway configured with Cognito authorizer
- Shared types exported and importable by other units

---

## Unit 2: Calorie Tracking

| Attribute | Value |
|-----------|-------|
| **Name** | calorie-tracking |
| **Branch** | unit/calorie-tracking |
| **Stories** | US-04, US-05, US-06, US-07, US-08, US-09, US-10, US-16 (8 stories) |
| **Effort** | L + M + M + S + M + M + L + M = Large |

### Scope
- Meal Service (Lambda: `calorie-app-meals`)
- AI Analysis Service (Lambda: `calorie-app-ai`)
- Image Service (Lambda: `calorie-app-images`)
- Meal Logging frontend module (photo upload, AI results, manual entry, custom foods)
- Nutrition Dashboard frontend module (daily progress, macros, goal tracking)
- Meal History frontend (view, edit, delete past meals)
- Weekly/Monthly charts (US-10 is v2 but US-08/09 are MVP)

### Delivers
- Complete meal logging flow (photo → AI analysis → confirm → save)
- Manual food entry with search and custom foods
- Full nutritional breakdown per meal
- Daily calorie and macro goal tracking with visual progress
- Meal history with edit/delete capability

### Dependencies
- **Foundation Unit** must be complete (auth, infra, shared types, offline sync)

### Exit Criteria
- User can upload meal photo and get AI nutrition estimate
- User can manually log food items with portion sizes
- Dashboard shows daily calorie/macro progress vs. goals
- Meal history displays with edit/delete functionality
- All data persists in DynamoDB and syncs offline

---

## Unit 3: Recipe Advisor

| Attribute | Value |
|-----------|-------|
| **Name** | recipe-advisor |
| **Branch** | unit/recipe-advisor |
| **Stories** | US-11, US-12, US-13, US-14, US-15 (5 stories) |
| **Effort** | L + S + XL + M + M = Large |

### Scope
- Recipe Service (Lambda: `calorie-app-recipes`)
- Ingredient Service (Lambda: `calorie-app-ingredients`)
- Ingredient Management frontend module (fridge photo, manual list, categories)
- Recipe Advisor frontend module (suggestions, details, filters, save)

### Delivers
- Fridge photo scanning with AI ingredient identification
- Manual ingredient inventory management
- AI-powered recipe suggestions based on ingredients + goals + preferences + time/skill
- Full recipe detail view with nutrition, steps, and metadata
- Recipe filtering by cooking time, skill level, dietary needs
- Save favorite recipes

### Dependencies
- **Foundation Unit** must be complete (auth, infra, shared types, offline sync)
- Uses AI Analysis Service from Calorie Tracking unit for fridge photo analysis (shared Lambda)

### Cross-Unit Contract
- Recipe Advisor calls `POST /ai/analyze/fridge` (owned by AI Analysis Service in Calorie Tracking unit)
- Contract: `{ imageKey: string, userId: string }` → `{ ingredients[]: { name, category, confidence } }`

### Exit Criteria
- User can scan fridge photo and get ingredient list
- User can manually manage ingredient inventory
- AI generates relevant recipe suggestions based on available ingredients
- Recipes display with full details, nutrition, and steps
- User can filter and save recipes

---

## Summary

| Unit | Stories | Effort | Dependencies | Merge Order |
|------|---------|--------|--------------|-------------|
| Foundation | 4 | Medium-Large | None | 1st |
| Calorie Tracking | 8 | Large | Foundation | 2nd |
| Recipe Advisor | 5 | Large | Foundation | 2nd (parallel with Calorie Tracking) |
