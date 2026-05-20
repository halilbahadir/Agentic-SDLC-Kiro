# Recipe Advisor Unit — NFR Requirements Plan

---

## 👥 Unit Team

| Member | Role | Responsibility |
|--------|------|---------------|
| awsHalil ⭐ | Developer | Story development (frontend) — Ingredient Mgmt Module, Recipe Advisor Module, AWS Bedrock integration (Lead) |
| halilbahadir | Tech Lead | Story development (backend) — Recipe Service, Ingredient Service |

> 📣 **awsHalil**: Invite halilbahadir for this session — Bedrock performance targets, DynamoDB capacity decisions, and Lambda configuration are backend concerns that benefit from Tech Lead input.

---

## 📐 Inherited NFRs from Foundation (Already Decided — Not Re-Asked)

The following NFRs are established at the project level and apply to Recipe Advisor without modification:

| NFR Area | Inherited Value |
|----------|----------------|
| Concurrent users | 100 (project-wide target) |
| DynamoDB capacity | On-demand (both IngredientsTable and RecipesTable) |
| Lambda runtime | Node.js 20.x, arm64 (Graviton2) |
| Lambda memory (standard) | 256MB for IngredientsFunction |
| Lambda memory (AI) | 512MB for RecipesFunction (Bedrock calls) |
| Lambda timeout (AI) | 60s hard ceiling — prompt must complete in < 30s (BR-RA-04) |
| API Gateway throttling | 50 req/s burst, 25 req/s sustained (shared) |
| Error rate target | < 1% of requests |
| Retry policy | 3 retries, exponential backoff (1s, 2s, 4s) |
| Availability | 99% uptime |
| Region | eu-west-1 (Ireland) |
| Environments | dev + prod |
| Encryption at rest | DynamoDB AES-256 (AWS managed) |
| Encryption in transit | HTTPS/TLS 1.2+ (CloudFront + API Gateway) |
| Logging | ERROR level only, structured JSON |
| Monitoring | CloudWatch (shared dashboard + alarms) |
| Offline cache budget | 30 days / ~50MB shared across all units |
| Cache TTL (stale data) | 24h for ingredient list and saved recipes |
| Responsive design | Mobile-first, 320px+ |
| Accessibility | WCAG 2.1 AA target |
| AI model | AWS Bedrock Claude 4.6 Sonnet (locked) |

**Recipe Advisor NFR questions focus only on what is NEW or SPECIFIC to this unit** — primarily Bedrock performance, ingredient/recipe data volumes, and UX loading behaviour for AI-heavy operations.

---

## NFR Requirements Steps

- [ ] Step 1: Analyze functional design artifacts for NFR implications
- [ ] Step 2: Collect and analyze answers to questions below
- [ ] Step 3: Generate `nfr-requirements.md`
- [ ] Step 4: Generate `tech-stack-decisions.md`
- [ ] Step 5: Present completion message and await approval

---

## Questions

Answer each question by filling in the `[Answer]:` tag below it.

---

### Section 1: Bedrock Performance Targets

**Q1** — The user story US-13 acceptance criteria states recipe suggestions must appear "within 15 seconds". Given the 30-second prompt complexity constraint (BR-RA-04), what is the acceptable end-to-end latency target for recipe generation (from "Get Recipes" tap to results displayed)?

```
A) < 10 seconds (aggressive — requires tight prompt engineering)
B) < 15 seconds (matches acceptance criteria exactly)
C) < 20 seconds (relaxed — more room for prompt complexity)
D) < 30 seconds (maximum — use full Lambda budget)
```
[Answer]: 

---

**Q2** — For fridge photo scanning (US-11), the acceptance criteria states ingredients should appear "within 10 seconds". What is the acceptable end-to-end latency target for fridge scan (from photo upload complete to ingredient list displayed)?

```
A) < 10 seconds (matches acceptance criteria exactly)
B) < 15 seconds (relaxed — fridge scan is less time-sensitive than recipe generation)
C) < 20 seconds (maximum acceptable)
```
[Answer]: 

---

**Q3** — When recipe generation is in progress (Bedrock call running), what loading UX is acceptable given the potentially long wait?

```
A) Simple spinner + "Generating recipes..." text — no progress indication
B) Animated progress bar with estimated time remaining (e.g., "~10 seconds")
C) Step-by-step status messages (e.g., "Analyzing your ingredients..." → "Finding recipes..." → "Almost done...")
D) Skeleton cards (placeholder recipe cards that fill in when ready)
```
[Answer]: 

---

### Section 2: Data Volume & Storage

**Q4** — The ingredient inventory is capped at 100 items (BR-RA-01). For the recipe suggestion Bedrock prompt, should the full ingredient list always be sent, or truncated if large?

```
A) Always send the full list (up to 100 items) — let Bedrock handle it
B) Send a maximum of 30 most-recently-added ingredients to keep prompt size manageable
C) Send a maximum of 50 ingredients, prioritising the most recently added
D) Let the user select which ingredients to include before generating
```
[Answer]: 

---

**Q5** — Saved recipes can accumulate over time. Is there a limit on how many recipes a user can save, or should the system handle unlimited saves?

```
A) No limit — store all saved recipes indefinitely
B) Soft limit: warn user at 50 saved recipes ("You have many saved recipes — consider removing some")
C) Hard limit: 100 saved recipes maximum (oldest must be deleted before saving new)
D) Hard limit: 200 saved recipes maximum
```
[Answer]: 

---

**Q6** — For the PouchDB cache, Recipe Advisor stores: ingredient list + saved recipes + recipe suggestion cache (14-day TTL). Given the shared 50MB budget, what is the estimated maximum size of Recipe Advisor's PouchDB footprint?

```
A) Small — ingredients (< 1MB) + saved recipes (< 2MB) + suggestion cache (< 1MB) = ~4MB total. No concern.
B) Medium — could reach 10–15MB if user has many saved recipes with full nutrition data. Monitor but no action needed.
C) Large — needs active management. Limit saved recipe cache to last 20 recipes in PouchDB (older ones fetched on demand).
D) Unknown — needs measurement after implementation.
```
[Answer]: 

---

### Section 3: Reliability & Error Handling

**Q7** — Bedrock is an external AI service with its own availability SLA. What is the acceptable degraded behaviour when Bedrock is unavailable (service outage, not timeout)?

```
A) Show error message only — no fallback
B) Show error + display cached recipe suggestions if available (14-day cache)
C) Show error + offer to navigate to saved recipes as an alternative
D) Both B and C — show cached suggestions if available, otherwise offer saved recipes
```
[Answer]: 

---

**Q8** — For ingredient CRUD operations (add/remove/update), what is the acceptable API response time target?

```
A) < 500ms (fast — matches profile CRUD from Foundation)
B) < 1 second
C) < 2 seconds
D) No specific target — these are background operations
```
[Answer]: 

---

**Q9** — The recipe suggestion cache has a 14-day TTL. Should there be a background refresh mechanism, or is on-demand refresh (user taps "Get Recipes") sufficient?

```
A) On-demand only — cache refreshes when user explicitly requests new recipes
B) Background refresh: if cache is > 7 days old and app is opened, silently refresh in background
C) Background refresh: if ingredients have changed since last generation, auto-refresh on app open
D) No caching at all — always call Bedrock fresh (override the 14-day decision)
```
[Answer]: 

---

### Section 4: Usability & Accessibility

**Q10** — For the ingredient list (potentially up to 100 items grouped by category), what rendering performance approach is needed?

```
A) Render all items at once — 100 items is manageable for React without virtualisation
B) Virtualise the list (react-window or similar) — only render visible items
C) Paginate by category — load one category at a time
D) Lazy load within categories — show first 10 per category, "Show more" per category
```
[Answer]: 

---

**Q11** — The fridge scan and recipe generation features are online-only. When the user is offline and tries to access these features, what is the required UX response time for the "offline" feedback?

```
A) Immediate (< 100ms) — check online status before any action, show inline disabled state
B) Attempt the action, show offline error within 1 second if it fails
C) Show a persistent offline banner at the top of the screen when offline (proactive, not reactive)
```
[Answer]: 

---

### Section 5: Security

**Q12** — Fridge photos and meal photos are uploaded to S3 with pre-signed URLs. Should fridge images be retained after ingredient identification, or deleted immediately?

```
A) Delete immediately after AI analysis completes (no retention)
B) Retain for 24 hours (allows retry if analysis fails), then auto-delete
C) Retain for 7 days (same as CloudWatch log retention in dev), then auto-delete
D) Retain indefinitely (same lifecycle as meal images)
```
[Answer]: 

---

**Q13** — The Bedrock prompt includes the user's ingredient list and dietary preferences. Is there any PII concern with sending this data to Bedrock?

```
A) No concern — ingredient names and dietary preferences are not PII
B) Minor concern — strip userId from prompt, use anonymous context only
C) Significant concern — requires data processing agreement review before using Bedrock
D) No concern, but log a note that Bedrock data handling follows AWS's standard data privacy terms
```
[Answer]: 

---
