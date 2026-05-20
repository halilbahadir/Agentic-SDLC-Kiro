# Unit of Work Plan

> 👥 **MULTI-DEVELOPER MODE**: Questions should be answered collaboratively 
> by the team. Involve relevant stakeholders for architectural and domain decisions.

---

## Context

Based on the Application Design, the system has:
- **6 backend services** (Auth, Meal, AI Analysis, Recipe, Ingredient, Image)
- **7 frontend modules** (Auth, Meal Logging, Nutrition Dashboard, Ingredient Mgmt, Recipe Advisor, Offline Sync, Shared UI)
- **2 developers**: halilbahadir (backend/Tech Lead) and awsHalil (frontend/AWS)
- **16 MVP stories** to implement

**Unit Definition**: Each unit is a vertical slice that owns stories end-to-end (frontend + backend + infrastructure), goes through its own full Construction cycle, and is assigned to a developer on its own branch.

---

## Clarifying Questions

### Question 1: Unit Decomposition Strategy
How should the system be split into units of work?

A) 2 feature units:
   - **Calorie Tracking Unit** — Auth + Meal Service + AI Analysis + Image Service + Auth/Meal/Dashboard/Offline Sync frontend modules + all CDK infrastructure (US-01 to US-10, US-16, US-17 — 12 stories)
   - **Recipe Advisor Unit** — Recipe Service + Ingredient Service + Ingredient/Recipe frontend modules (US-11 to US-15 — 5 stories)

B) 3 units with foundation:
   - **Foundation Unit** — Shared types, CDK infra (API Gateway, DynamoDB, S3, Cognito, CloudFront), Auth Service + Auth Module, project scaffolding, Offline Sync (US-01, US-02, US-03, US-17 — 4 stories)
   - **Calorie Tracking Unit** — Meal Service + AI Analysis + Image Service + Meal Logging/Dashboard frontend (US-04 to US-10, US-16 — 8 stories)
   - **Recipe Advisor Unit** — Recipe Service + Ingredient Service + Ingredient/Recipe frontend (US-11 to US-15 — 5 stories)

C) 5 fine-grained units:
   - **Auth & Foundation Unit** — Auth Service + Auth Module + shared infra + Offline Sync (US-01, US-02, US-03, US-17 — 4 stories)
   - **Meal Logging Unit** — Meal Service + AI Analysis + Image Service + Meal Logging Module (US-04, US-05, US-06, US-07 — 4 stories)
   - **Nutrition Tracking Unit** — Nutrition Dashboard Module + Meal History (US-08, US-09, US-10, US-16 — 4 stories)
   - **Ingredient Management Unit** — Ingredient Service + Ingredient Module (US-11, US-12 — 2 stories)
   - **Recipe Advisor Unit** — Recipe Service + Recipe Advisor Module (US-13, US-14, US-15 — 3 stories)

D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

### Question 2: Code Organization
How should the code be organized in the repository?

A) Monorepo with npm workspaces — shared dependencies, independent builds per unit:
   ```
   packages/
     shared/          (types, utilities)
     infrastructure/  (CDK stacks)
     frontend/        (React app)
     backend/
       auth/
       meals/
       recipes/
       ingredients/
       ai-analysis/
       images/
   ```

B) Flat monorepo — simpler structure, single package.json:
   ```
   src/
     frontend/    (React app)
     backend/     (all Lambda handlers)
     shared/      (types, utilities)
   infra/         (CDK)
   ```

C) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Generation Steps

After questions are answered, the following steps will be executed:

- [x] Step 1: Define unit boundaries based on decomposition strategy
- [x] Step 2: Assign stories to units
- [x] Step 3: Map dependencies between units
- [x] Step 4: Define contracts for cross-unit interfaces
- [x] Step 5: Suggest developer assignment (MULTIDEV-04)
- [x] Step 6: Determine merge order
- [x] Step 7: Generate unit-of-work.md
- [x] Step 8: Generate unit-of-work-dependency.md
- [x] Step 9: Generate unit-of-work-story-map.md
- [x] Step 10: Validate completeness

---
