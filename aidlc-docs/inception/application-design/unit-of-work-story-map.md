# Unit of Work — Story Map

## Story Assignment

| Story ID | Story Title | Unit | Persona | Effort | MVP |
|----------|-------------|------|---------|--------|-----|
| US-01 | User Registration | Foundation | Both | S | ✅ |
| US-02 | User Login & Session | Foundation | Both | S | ✅ |
| US-03 | Profile Setup | Foundation | Both | M | ✅ |
| US-17 | Offline Access & Cloud Sync | Foundation | Both | L | ✅ |
| US-04 | Upload Meal Photo | Calorie Tracking | Alex | L | ✅ |
| US-05 | Confirm/Correct AI ID | Calorie Tracking | Alex | M | ✅ |
| US-06 | Manual Food Entry | Calorie Tracking | Alex | M | ✅ |
| US-07 | Custom Food Item | Calorie Tracking | Alex | S | ✅ |
| US-08 | Nutrition Breakdown | Calorie Tracking | Alex | M | ✅ |
| US-09 | Daily Goal Tracking | Calorie Tracking | Alex | M | ✅ |
| US-10 | Weekly/Monthly Charts | Calorie Tracking | Alex | L | ❌ v2 |
| US-16 | Meal History & Edit | Calorie Tracking | Alex | M | ✅ |
| US-11 | Fridge Photo Scan | Recipe Advisor | Sam | L | ✅ |
| US-12 | Manual Ingredient List | Recipe Advisor | Sam | S | ✅ |
| US-13 | AI Recipe Suggestions | Recipe Advisor | Sam | XL | ✅ |
| US-14 | Recipe Details View | Recipe Advisor | Sam | M | ✅ |
| US-15 | Recipe Filters | Recipe Advisor | Sam | M | ✅ |

## Per-Unit Story Summary

### Foundation (4 stories)
| Story | What It Delivers |
|-------|-----------------|
| US-01 | User can register with email/password |
| US-02 | User can login, stay logged in, reset password |
| US-03 | User can set calorie goals, dietary preferences, skill level |
| US-17 | App works offline, syncs when reconnected |

### Calorie Tracking (8 stories — 7 MVP + 1 v2)
| Story | What It Delivers |
|-------|-----------------|
| US-04 | Upload meal photo → AI identifies foods |
| US-05 | Review and correct AI food identification |
| US-06 | Manually search and add food items |
| US-07 | Create custom food items with nutrition data |
| US-08 | View full nutritional breakdown per meal |
| US-09 | Daily calorie/macro progress vs. goals |
| US-10 | Weekly/monthly trend charts (**v2**) |
| US-16 | View, edit, delete past meals |

### Recipe Advisor (5 stories)
| Story | What It Delivers |
|-------|-----------------|
| US-11 | Scan fridge photo → AI identifies ingredients |
| US-12 | Manually add/remove ingredients from inventory |
| US-13 | AI generates recipe suggestions based on ingredients + goals + preferences |
| US-14 | View full recipe details (steps, nutrition, metadata) |
| US-15 | Filter recipes by time, skill, dietary needs |

## Developer Assignment (MULTIDEV-04 Suggestion)

| Unit | Suggested Developer | Rationale |
|------|-------------------|-----------|
| Foundation | halilbahadir | Tech Lead sets up infrastructure, auth patterns, and shared types that define project standards |
| Calorie Tracking | halilbahadir | Backend-heavy (3 Lambda services, AI integration, complex data aggregation) |
| Recipe Advisor | awsHalil | AWS expertise for Bedrock recipe generation, frontend-heavy recipe UI, ingredient management |

### Workload Summary

| Developer | Units | Total Stories | Effort Profile |
|-----------|-------|---------------|----------------|
| halilbahadir | Foundation + Calorie Tracking | 12 stories | Sequential: Foundation first, then Calorie Tracking |
| awsHalil | Recipe Advisor | 5 stories | Starts after Foundation complete |

### Sequencing

```
Timeline:
─────────────────────────────────────────────────────────────
halilbahadir:  [Foundation (4 stories)] → [Calorie Tracking (8 stories)]
awsHalil:     [waiting...............] → [Recipe Advisor (5 stories)   ]
─────────────────────────────────────────────────────────────
                                        ^ Foundation complete
                                          Both start feature units
```

**Note**: awsHalil can contribute to Foundation unit collaboratively (e.g., frontend scaffolding, shared UI components) to reduce wait time, then switch to Recipe Advisor branch once Foundation merges.
