# Requirements Verification Questions

Please answer the following questions to help clarify the requirements for your Calorie Counter & Recipe Advisor application. Fill in the letter choice after each [Answer]: tag.

---

## Question 1
What platform should this application run on?

A) Web application (browser-based)
B) Mobile application (iOS/Android)
C) Both web and mobile
D) Desktop application
E) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 2
How should users authenticate and manage their accounts?

A) Email/password registration
B) Social login (Google, Apple, Facebook)
C) Both email/password and social login
D) No authentication needed (local-only app)
E) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 3
For the AI calorie estimation, which approach do you prefer?

A) Use a cloud AI service (e.g., AWS Bedrock, OpenAI API) — more accurate, requires internet
B) Use on-device AI model — works offline, less accurate
C) Hybrid — cloud AI when online, basic local estimation when offline
D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 4
How should meal data be stored?

A) Cloud database (accessible from multiple devices, requires account)
B) Local storage only (single device, no sync)
C) Both — local with optional cloud sync
D) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## Question 5
What level of calorie tracking detail do you need?

A) Just total calories per meal
B) Calories + macronutrients (protein, carbs, fat)
C) Full nutritional breakdown (calories, macros, vitamins, minerals)
D) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## Question 6
Should the app track daily/weekly goals and progress?

A) Yes — set daily calorie goals and track progress over time with charts
B) Yes — simple daily tracking without historical charts
C) No — just estimate calories per meal, no goal tracking
D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 7
For the "what to cook" recipe advisor, what should it consider?

A) Only available ingredients (from fridge photos or manual list)
B) Available ingredients + calorie/nutrition goals
C) Available ingredients + calorie goals + dietary preferences (vegan, keto, etc.)
D) All above + cooking time and skill level preferences
E) Other (please describe after [Answer]: tag below)

[Answer]: D

---

## Question 8
How should recipe suggestions be presented?

A) Simple text-based recipe list with ingredients and steps
B) Rich recipe cards with images, prep time, difficulty level
C) Full meal planning (suggest breakfast, lunch, dinner combinations)
D) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## Question 9
What is the target cloud provider for backend services?

A) AWS (Lambda, DynamoDB, Bedrock, S3, etc.)
B) Google Cloud (Cloud Functions, Firestore, Vertex AI)
C) Azure (Functions, CosmosDB, Azure OpenAI)
D) No preference — choose the best fit
E) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 10
What is the primary language/framework preference for development?

A) TypeScript/React (frontend) + Node.js/TypeScript (backend)
B) TypeScript/React Native (mobile) + Node.js (backend)
C) Python (backend/AI) + React (frontend)
D) Flutter (mobile) + Python (backend)
E) No preference — choose the best fit for the use case
F) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 11: Security Extensions
Should security extension rules be enforced for this project?

A) Yes — enforce all SECURITY rules as blocking constraints (recommended for production-grade applications)
B) No — skip all SECURITY rules (suitable for PoCs, prototypes, and experimental projects)
C) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Question 12: Multi-Developer Extension
Is this a multi-developer project with team-based parallel development?

A) Yes — enforce multi-developer rules: team definition, unit assignment, branching strategy, deviation gates, and integration workflow
B) No — single developer workflow, skip all multi-developer rules (standard AI-DLC behavior)
C) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 13: Property-Based Testing Extension
Should property-based testing (PBT) rules be enforced for this project?

A) Yes — enforce all PBT rules as blocking constraints (recommended for projects with business logic, data transformations, serialization, or stateful components)
B) Partial — enforce PBT rules only for pure functions and serialization round-trips (suitable for projects with limited algorithmic complexity)
C) No — skip all PBT rules (suitable for simple CRUD applications, UI-only projects, or thin integration layers with no significant business logic)
D) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## Question 14: Toolchain Integration Extension
What external tools will be used in this project?

A) Project Management — Jira, Linear, Azure DevOps (sync stories, track issues, sprint planning)
B) Version Control Platform — GitHub, GitLab, Bitbucket (branches, PRs/MRs, code review via MCP)
C) Documentation — Confluence, Notion (publish design docs, architecture decisions)
D) Architecture & Diagramming — Miro, DrawIO, Mermaid AI (visual design, architecture diagrams)
E) CI/CD — GitHub Actions, GitLab CI, Jenkins, Bamboo, AWS CodePipeline (build pipelines)
F) None — all work stays within AI-DLC local files only
G) Other (please describe after [Answer]: tag below)

[Answer]: B, GitHub

---
