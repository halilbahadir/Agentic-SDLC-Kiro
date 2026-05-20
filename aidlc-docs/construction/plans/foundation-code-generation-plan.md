# Foundation Unit — Code Generation Plan

---

## 👥 Unit Team

| Member | Role | Responsibility |
|--------|------|---------------|
| halilbahadir ⭐ | Tech Lead | Story development (backend) — Auth Service, CDK infra, shared types, API Gateway (Lead) |
| awsHalil | Developer | Story development (frontend) — Auth Module, Offline Sync Module, Shared UI, CloudFront/S3 |

---

## Unit Context

| Attribute | Value |
|-----------|-------|
| **Unit** | Foundation |
| **Stories** | US-01, US-02, US-03, US-17 |
| **Workspace Root** | /Users/halilb/AWSWorks/Agentic-SDLC-Kiro |
| **Project Type** | Greenfield, flat monorepo |
| **Dependencies** | None (Foundation is the base unit) |
| **Provides To** | Calorie Tracking, Recipe Advisor (shared types, infra, auth, offline sync) |

## Target File Structure

```
src/
  frontend/
    index.html
    main.tsx
    App.tsx
    vite-env.d.ts
    components/
      shared/
        NavigationBar.tsx
        LoadingSpinner.tsx
        ErrorMessage.tsx
        Toast.tsx
        ImageUpload.tsx
        SearchInput.tsx
    modules/
      auth/
        LoginPage.tsx
        RegisterPage.tsx
        ForgotPasswordPage.tsx
        ProfileSetupWizard.tsx
      offline-sync/
        SyncStatusIndicator.tsx
    contexts/
      AuthProvider.tsx
      SyncProvider.tsx
      ThemeProvider.tsx
    hooks/
      useApi.ts
    services/
      api-client.ts
      pouchdb-service.ts
    styles/
      index.css (Tailwind imports)
  backend/
    shared/
      handler.ts
      dynamodb-client.ts
      validation.ts
      logger.ts
      types.ts
    auth/
      index.ts (Lambda handler)
      auth-service.ts
  shared/
    types.ts (cross-layer shared types)
infra/
  bin/
    app.ts
  lib/
    auth-stack.ts
    storage-stack.ts
    api-stack.ts
    hosting-stack.ts
    monitoring-stack.ts
  config/
    dev.ts
    prod.ts
package.json
tsconfig.json
vite.config.ts
tailwind.config.js
postcss.config.js
.gitignore
README.md
```

---

## Code Generation Steps

### Phase A: Project Scaffolding (Greenfield Setup)

- [ ] **Step 1**: Create root configuration files
  - `package.json` (dependencies, scripts)
  - `tsconfig.json` (TypeScript config)
  - `vite.config.ts` (Vite build config)
  - `tailwind.config.js` (Tailwind CSS config)
  - `postcss.config.js` (PostCSS for Tailwind)
  - `.gitignore`
  - **Stories**: Foundation for all

- [ ] **Step 2**: Create shared types
  - `src/shared/types.ts` (NutritionData, MealItem, Ingredient, Recipe, UserProfile, etc.)
  - **Stories**: US-01, US-02, US-03 (profile types), US-17 (sync types)

### Phase B: Backend — Shared Utilities

- [ ] **Step 3**: Create backend shared utilities
  - `src/backend/shared/handler.ts` (Lambda handler wrapper)
  - `src/backend/shared/dynamodb-client.ts` (DynamoDB Document Client wrapper)
  - `src/backend/shared/validation.ts` (input validation utility)
  - `src/backend/shared/logger.ts` (structured ERROR logging)
  - `src/backend/shared/types.ts` (backend-specific types)
  - **Stories**: Foundation for all backend services

### Phase C: Backend — Auth Service

- [ ] **Step 4**: Create Auth Service business logic
  - `src/backend/auth/auth-service.ts` (register, confirm, login, refresh, forgotPassword, confirmPassword, getProfile, updateProfile)
  - **Stories**: US-01, US-02, US-03

- [ ] **Step 5**: Create Auth Service Lambda handler
  - `src/backend/auth/index.ts` (route requests to auth-service methods)
  - **Stories**: US-01, US-02, US-03

- [ ] **Step 6**: Create Auth Service unit tests
  - `src/backend/auth/auth-service.test.ts`
  - **Stories**: US-01, US-02, US-03

### Phase D: Frontend — Core Setup

- [ ] **Step 7**: Create frontend entry point and app shell
  - `src/frontend/index.html`
  - `src/frontend/main.tsx`
  - `src/frontend/App.tsx` (Router, providers, layout)
  - `src/frontend/vite-env.d.ts`
  - `src/frontend/styles/index.css` (Tailwind imports)
  - **Stories**: Foundation for all frontend

- [ ] **Step 8**: Create context providers
  - `src/frontend/contexts/AuthProvider.tsx`
  - `src/frontend/contexts/SyncProvider.tsx`
  - `src/frontend/contexts/ThemeProvider.tsx`
  - **Stories**: US-02 (auth state), US-17 (sync state)

- [ ] **Step 9**: Create API client and services
  - `src/frontend/services/api-client.ts` (axios instance with interceptors)
  - `src/frontend/services/pouchdb-service.ts` (PouchDB local database)
  - `src/frontend/hooks/useApi.ts` (reusable data fetching hook)
  - **Stories**: US-17 (offline sync), US-02 (auth tokens)

### Phase E: Frontend — Shared UI Components

- [ ] **Step 10**: Create shared UI components
  - `src/frontend/components/shared/NavigationBar.tsx`
  - `src/frontend/components/shared/LoadingSpinner.tsx`
  - `src/frontend/components/shared/ErrorMessage.tsx`
  - `src/frontend/components/shared/Toast.tsx`
  - `src/frontend/components/shared/ImageUpload.tsx`
  - `src/frontend/components/shared/SearchInput.tsx`
  - **Stories**: Foundation for all UI

### Phase F: Frontend — Auth Module

- [ ] **Step 11**: Create auth pages
  - `src/frontend/modules/auth/LoginPage.tsx`
  - `src/frontend/modules/auth/RegisterPage.tsx`
  - `src/frontend/modules/auth/ForgotPasswordPage.tsx`
  - `src/frontend/modules/auth/ProfileSetupWizard.tsx`
  - **Stories**: US-01, US-02, US-03

- [ ] **Step 12**: Create auth module tests
  - `src/frontend/modules/auth/LoginPage.test.tsx`
  - `src/frontend/modules/auth/RegisterPage.test.tsx`
  - **Stories**: US-01, US-02

### Phase G: Frontend — Offline Sync Module

- [ ] **Step 13**: Create offline sync UI
  - `src/frontend/modules/offline-sync/SyncStatusIndicator.tsx`
  - **Stories**: US-17

### Phase H: Infrastructure (CDK)

- [ ] **Step 14**: Create CDK project structure
  - `infra/bin/app.ts` (CDK app entry)
  - `infra/config/dev.ts` (dev environment config)
  - `infra/config/prod.ts` (prod environment config)
  - `infra/package.json` (CDK dependencies)
  - `infra/tsconfig.json`
  - `infra/cdk.json`
  - **Stories**: Foundation infrastructure

- [ ] **Step 15**: Create CDK stacks
  - `infra/lib/auth-stack.ts` (Cognito)
  - `infra/lib/storage-stack.ts` (DynamoDB + S3)
  - `infra/lib/api-stack.ts` (API Gateway + Lambdas)
  - `infra/lib/hosting-stack.ts` (CloudFront + S3 frontend)
  - `infra/lib/monitoring-stack.ts` (CloudWatch dashboard + alarms)
  - **Stories**: Foundation infrastructure

### Phase I: Documentation & Deployment

- [ ] **Step 16**: Create README and documentation
  - `README.md` (project overview, setup instructions, deployment guide)
  - `aidlc-docs/construction/foundation/code/code-summary.md` (generated code summary)
  - **Stories**: All

---

## Story Traceability

| Story | Steps Implementing |
|-------|-------------------|
| US-01 (Registration) | Steps 2, 4, 5, 6, 8, 11, 12 |
| US-02 (Login/Session) | Steps 2, 4, 5, 6, 8, 9, 11, 12 |
| US-03 (Profile Setup) | Steps 2, 4, 5, 6, 11 |
| US-17 (Offline Sync) | Steps 2, 8, 9, 13 |

---
