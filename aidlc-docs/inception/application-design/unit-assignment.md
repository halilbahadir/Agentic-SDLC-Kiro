# Unit Assignment

## Story Assignment Strategy: B
Stories are assigned by layer within story (specialized teams: frontend/backend).

## Unit Teams

| Unit | Team Members | Lead |
|------|-------------|------|
| Foundation | halilbahadir, awsHalil | halilbahadir |
| Calorie Tracking | halilbahadir, awsHalil | halilbahadir |
| Recipe Advisor | halilbahadir, awsHalil | awsHalil |

## Responsibility Matrix

### Foundation

| Member | Role | Responsibility | Scope |
|--------|------|---------------|-------|
| halilbahadir ⭐ | Tech Lead | Story development (backend) | Auth Service, CDK infra, shared types, API Gateway |
| awsHalil | Developer | Story development (frontend) | Auth Module, Offline Sync Module, Shared UI, CloudFront/S3 |

### Calorie Tracking

| Member | Role | Responsibility | Scope |
|--------|------|---------------|-------|
| halilbahadir ⭐ | Tech Lead | Story development (backend) | Meal Service, AI Analysis Service, Image Service, DynamoDB |
| awsHalil | Developer | Story development (frontend) | Meal Logging Module, Nutrition Dashboard Module |

### Recipe Advisor

| Member | Role | Responsibility | Scope |
|--------|------|---------------|-------|
| awsHalil ⭐ | Developer | Story development (frontend) | Ingredient Mgmt Module, Recipe Advisor Module, AWS Bedrock integration |
| halilbahadir | Tech Lead | Story development (backend) | Recipe Service, Ingredient Service |

## Workload Balance

| Developer | Units (as team member) | Estimated Stories | Responsibility |
|-----------|----------------------|-------------------|----------------|
| halilbahadir | Foundation + Calorie Tracking + Recipe Advisor | 17 (backend portions) | Backend across all units |
| awsHalil | Foundation + Calorie Tracking + Recipe Advisor | 17 (frontend portions) | Frontend across all units |

> ⚖️ Balanced: Both developers participate in all 3 units with complementary responsibilities.

## Merge Order (dependency-based)

| Order | Unit | Depends On | Can merge after |
|-------|------|-----------|-----------------|
| 1 | Foundation | — | Immediately |
| 2 | Calorie Tracking | Foundation | Foundation merged |
| 2 | Recipe Advisor | Foundation | Foundation merged |

## Branch Structure

```
main (production)
 └── develop (integration)
      └── unit/foundation (unit integration branch)
           └── feature/US-01/backend (halilbahadir)
           └── feature/US-01/frontend (awsHalil)
           └── feature/US-02/backend (halilbahadir)
           └── feature/US-02/frontend (awsHalil)
           └── feature/US-03/backend (halilbahadir)
           └── feature/US-03/frontend (awsHalil)
           └── feature/US-17/backend (halilbahadir)
           └── feature/US-17/frontend (awsHalil)
      └── unit/calorie-tracking
           └── feature/US-04/backend
           └── feature/US-04/frontend
           └── ...
      └── unit/recipe-advisor
           └── feature/US-11/backend
           └── feature/US-11/frontend
           └── ...
```
