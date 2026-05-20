# Foundation Unit — Functional Design Plan

---

## 👥 Unit Team

| Member | Role | Responsibility |
|--------|------|---------------|
| halilbahadir ⭐ | Tech Lead | Story development (backend) — Auth Service, CDK infra, shared types, API Gateway (Lead) |
| awsHalil | Developer | Story development (frontend) — Auth Module, Offline Sync Module, Shared UI, CloudFront/S3 |

> 📣 **halilbahadir**: This is a collaborative session. Please invite all team members 
> listed above to answer these questions together. Each member brings expertise relevant 
> to their responsibility — collective input produces better designs.

---

## 📐 Inherited Design Principles

The following principles were established during Application Design and apply to this unit:

- **Architecture**: Service-per-feature (separate Lambda per feature area)
- **API Style**: REST with resource-based URLs
- **Auth**: AWS Cognito (managed, email/password)
- **Frontend State**: React Context + useReducer
- **Offline Sync**: PouchDB/CouchDB pattern
- **API Communication**: axios/fetch with custom hooks
- **Database**: Hybrid DynamoDB (single table for related entities, separate for independent)
- **Error Format**: `{ error: { code, message, details } }`
- **Logging**: Structured JSON (service, method, userId, timestamp, duration)
- **Auth at Gateway**: Cognito JWT validation at API Gateway level

---

## Unit Context

**Unit**: Foundation  
**Branch**: unit/foundation  
**Stories**: US-01 (Registration), US-02 (Login/Session), US-03 (Profile Setup), US-17 (Offline & Cloud Sync)  
**Scope**: Auth Service, Auth Module, Offline Sync, Shared Types, CDK Infrastructure, Shared UI

---

## Clarifying Questions

### Question 1: User Profile Data Model
What additional profile fields should be stored beyond what's in the stories?

A) Minimal — only what stories require: email, dailyCalorieTarget, macroTargets, dietaryPreferences[], cookingSkillLevel
B) Extended — add: displayName, timezone, weightGoal, height, weight, age, activityLevel (for better calorie recommendations)
C) Other (please describe after [Answer]: tag below)

[Answer]: B

---

### Question 2: Password Policy
What password requirements should be enforced?

A) Basic — minimum 8 characters, at least 1 number
B) Standard — minimum 8 characters, uppercase + lowercase + number + special character
C) Cognito default — minimum 8 characters, configurable via Cognito User Pool settings
D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

### Question 3: Offline Sync Conflict Resolution
When the same record is modified on two devices while offline, how should conflicts be resolved?

A) Last-write-wins (simpler, based on timestamp)
B) Server-wins (cloud version always takes priority)
C) Prompt user to choose (show both versions)
D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

### Question 4: Session Duration
How long should user sessions last before requiring re-login?

A) 7 days (refresh token expires after 7 days of inactivity)
B) 30 days (refresh token expires after 30 days)
C) Never expire (refresh token valid until explicit logout)
D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Generation Steps

After questions are answered, the following steps will be executed:

- [x] Step 1: Define domain entities (User, UserProfile, Session)
- [x] Step 2: Define business rules (registration validation, login flow, profile constraints)
- [x] Step 3: Define business logic model (auth flows, sync algorithm, offline queue)
- [x] Step 4: Define frontend components (Auth pages, Profile wizard, Sync indicator, Shared UI)
- [x] Step 5: Validate completeness against stories US-01, US-02, US-03, US-17

---
