# Deployment Architecture — Foundation Unit

## Architecture Diagram

```
                    eu-west-1 (Ireland)
+------------------------------------------------------------------+
|                                                                    |
|  +--------------------+         +--------------------+            |
|  |   CloudFront       |         |   API Gateway      |            |
|  |   Distribution     |         |   (REST API)       |            |
|  |   (SPA Hosting)    |         |   + Cognito Auth   |            |
|  +--------------------+         +--------------------+            |
|          |                              |                         |
|          v                              v                         |
|  +--------------------+    +----------------------------------+   |
|  |   S3 Bucket        |    |   Lambda Functions (Node.js 20)  |   |
|  |   (Frontend)       |    |   - auth (256MB, 30s)            |   |
|  |   - index.html     |    |   - meals (256MB, 30s)           |   |
|  |   - assets/        |    |   - ai-analysis (512MB, 60s)     |   |
|  +--------------------+    |   - recipes (512MB, 60s)          |   |
|                            |   - ingredients (256MB, 30s)      |   |
|                            |   - images (256MB, 30s)           |   |
|                            +----------------------------------+   |
|                                     |       |       |             |
|                    +----------------+-------+-------+------+      |
|                    |                |                |      |      |
|                    v                v                v      v      |
|  +-------------+  +-------------+  +--------+  +--------+        |
|  |  Cognito    |  |  DynamoDB   |  |   S3   |  |Bedrock |        |
|  |  User Pool  |  |  (3 tables) |  | Images |  | (AI)   |        |
|  +-------------+  +-------------+  +--------+  +--------+        |
|                    | MealsNutrition|                               |
|                    | Ingredients   |                               |
|                    | Recipes       |                               |
|                    +---------------+                               |
|                                                                    |
|  +--------------------+                                           |
|  |   CloudWatch       |                                           |
|  |   - Logs           |                                           |
|  |   - Metrics        |                                           |
|  |   - Dashboard      |                                           |
|  |   - Alarms → SNS   |                                           |
|  +--------------------+                                           |
|                                                                    |
+------------------------------------------------------------------+
```

## CDK Project Structure

```
infra/
  bin/
    app.ts                    # CDK app entry point
  lib/
    auth-stack.ts             # Cognito User Pool + Client
    storage-stack.ts          # DynamoDB tables + S3 buckets
    api-stack.ts              # API Gateway + Lambda functions
    hosting-stack.ts          # CloudFront + S3 frontend
    monitoring-stack.ts       # CloudWatch dashboard + alarms
  config/
    dev.ts                    # Dev environment config
    prod.ts                   # Prod environment config
  cdk.json                    # CDK configuration
  tsconfig.json               # TypeScript config for CDK
```

## Environment Configuration Pattern

```typescript
// infra/config/dev.ts
export const devConfig = {
  env: 'dev',
  region: 'eu-west-1',
  lambda: { memorySize: 256, timeout: 30 },
  aiLambda: { memorySize: 512, timeout: 60 },
  dynamodb: { pointInTimeRecovery: false },
  s3: { versioned: false, lifecycleDays: 90 },
  cloudwatch: { logRetentionDays: 7 },
  alarms: { enabled: false },
};

// infra/config/prod.ts
export const prodConfig = {
  env: 'prod',
  region: 'eu-west-1',
  lambda: { memorySize: 256, timeout: 30 },
  aiLambda: { memorySize: 512, timeout: 60 },
  dynamodb: { pointInTimeRecovery: true },
  s3: { versioned: true, lifecycleDays: undefined },
  cloudwatch: { logRetentionDays: 30 },
  alarms: { enabled: true, email: 'alerts@example.com' },
};
```

## Stack Dependency Graph

```
AuthStack (independent)
StorageStack (independent)
     \         /
      \       /
       v     v
      ApiStack (depends on Auth + Storage)
         |
         v
   MonitoringStack (depends on Api)

HostingStack (independent — deploys frontend separately)
```

## Resource Naming Convention

| Resource Type | Pattern | Example |
|---------------|---------|---------|
| DynamoDB Table | `calorie-app-{env}-{name}` | `calorie-app-dev-meals-nutrition` |
| S3 Bucket | `calorie-app-{env}-{name}-{account-id}` | `calorie-app-dev-images-123456789` |
| Lambda Function | `calorie-app-{env}-{service}` | `calorie-app-dev-auth` |
| API Gateway | `calorie-app-{env}-api` | `calorie-app-dev-api` |
| CloudFront | (auto-generated) | `d1234abcdef.cloudfront.net` |
| Cognito Pool | `calorie-app-{env}-users` | `calorie-app-dev-users` |

## Security Configuration

### S3 Bucket Policies
- Frontend bucket: CloudFront OAI access only (no public access)
- Images bucket: Lambda role access only (pre-signed URLs for client upload/download)

### API Gateway CORS
```
Access-Control-Allow-Origin: [CloudFront distribution URL]
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
```

### Lambda Environment Variables
| Variable | Source | Description |
|----------|--------|-------------|
| `MEALS_TABLE_NAME` | StorageStack output | DynamoDB table name |
| `INGREDIENTS_TABLE_NAME` | StorageStack output | DynamoDB table name |
| `RECIPES_TABLE_NAME` | StorageStack output | DynamoDB table name |
| `IMAGES_BUCKET_NAME` | StorageStack output | S3 bucket name |
| `USER_POOL_ID` | AuthStack output | Cognito User Pool ID |
| `USER_POOL_CLIENT_ID` | AuthStack output | Cognito Client ID |
| `REGION` | CDK context | AWS region |
| `ENV` | CDK context | Environment name |
