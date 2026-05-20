# Infrastructure Design — Foundation Unit

## Deployment Configuration

| Setting | Value |
|---------|-------|
| **Region** | eu-west-1 (Ireland) |
| **Environments** | dev, prod |
| **Custom Domain** | None (default CloudFront/API Gateway URLs) |
| **IaC** | AWS CDK v2 (TypeScript) |
| **Stack Naming** | `CalorieApp-{Env}-{StackName}` (e.g., `CalorieApp-Dev-AuthStack`) |

---

## CDK Stack Structure

### Stack 1: AuthStack
**Name**: `CalorieApp-{Env}-Auth`  
**Resources**:

| Resource | Type | Configuration |
|----------|------|---------------|
| UserPool | `aws_cognito.UserPool` | Email sign-in, standard password policy, email verification |
| UserPoolClient | `aws_cognito.UserPoolClient` | Auth flows: USER_PASSWORD_AUTH, REFRESH_TOKEN_AUTH |

**Key Settings**:
- Self sign-up: enabled
- Email verification: required (code-based)
- Password policy: min 8, uppercase, lowercase, number, special char
- Refresh token validity: 30 days
- Access token validity: 1 hour
- Account recovery: email only

**Outputs** (exported for other stacks):
- `UserPoolId`
- `UserPoolClientId`
- `UserPoolArn`

---

### Stack 2: StorageStack
**Name**: `CalorieApp-{Env}-Storage`  
**Resources**:

| Resource | Type | Configuration |
|----------|------|---------------|
| MealsNutritionTable | `aws_dynamodb.Table` | PK: `pk` (String), SK: `sk` (String), on-demand, PITR enabled |
| IngredientsTable | `aws_dynamodb.Table` | PK: `pk` (String), SK: `sk` (String), on-demand |
| RecipesTable | `aws_dynamodb.Table` | PK: `pk` (String), SK: `sk` (String), on-demand |
| ImagesBucket | `aws_s3.Bucket` | Private, versioned, lifecycle (delete after 90 days for dev) |

**MealsNutritionTable GSIs**:
- GSI1: `gsi1pk` (String) + `gsi1sk` (String) — for date-range queries

**ImagesBucket Settings**:
- Block all public access
- CORS: allow PUT from app origin (for pre-signed uploads)
- Lifecycle: dev env deletes objects after 90 days
- Encryption: SSE-S3 (default)

**Outputs**:
- `MealsNutritionTableName`, `MealsNutritionTableArn`
- `IngredientsTableName`, `IngredientsTableArn`
- `RecipesTableName`, `RecipesTableArn`
- `ImagesBucketName`, `ImagesBucketArn`

---

### Stack 3: ApiStack
**Name**: `CalorieApp-{Env}-Api`  
**Dependencies**: AuthStack, StorageStack  
**Resources**:

| Resource | Type | Configuration |
|----------|------|---------------|
| RestApi | `aws_apigateway.RestApi` | REST API with Cognito authorizer |
| CognitoAuthorizer | `aws_apigateway.CognitoUserPoolsAuthorizer` | Validates JWT from AuthStack UserPool |
| AuthFunction | `aws_lambda.Function` | Node.js 20, 256MB, 30s timeout |
| MealsFunction | `aws_lambda.Function` | Node.js 20, 256MB, 30s timeout |
| AiAnalysisFunction | `aws_lambda.Function` | Node.js 20, 512MB, 60s timeout (AI calls take longer) |
| RecipesFunction | `aws_lambda.Function` | Node.js 20, 512MB, 60s timeout |
| IngredientsFunction | `aws_lambda.Function` | Node.js 20, 256MB, 30s timeout |
| ImagesFunction | `aws_lambda.Function` | Node.js 20, 256MB, 30s timeout |

**API Gateway Settings**:
- Throttling: 50 req/s burst, 25 req/s sustained
- CORS: enabled for all routes
- Stage: `{env}` (dev or prod)
- Deploy options: stage logging disabled (use Lambda logs)

**Lambda Common Settings**:
- Runtime: Node.js 20.x
- Architecture: arm64 (Graviton2 — cheaper, faster)
- Bundling: esbuild (tree-shaking, minification)
- Environment variables: table names, bucket name, region, user pool ID

**Outputs**:
- `ApiUrl` (e.g., `https://abc123.execute-api.eu-west-1.amazonaws.com/dev`)

---

### Stack 4: HostingStack
**Name**: `CalorieApp-{Env}-Hosting`  
**Resources**:

| Resource | Type | Configuration |
|----------|------|---------------|
| FrontendBucket | `aws_s3.Bucket` | Private, static website hosting via CloudFront |
| Distribution | `aws_cloudfront.Distribution` | S3 origin, SPA routing (error pages → index.html) |
| OriginAccessIdentity | `aws_cloudfront.OAI` | S3 access for CloudFront only |

**CloudFront Settings**:
- Default root object: `index.html`
- Error pages: 403/404 → `/index.html` (SPA routing)
- Cache policy: CachingOptimized for static assets
- Price class: PriceClass_100 (cheapest — US, Canada, Europe)
- Viewer protocol: redirect HTTP to HTTPS

**Outputs**:
- `DistributionUrl` (e.g., `https://d1234abcdef.cloudfront.net`)
- `FrontendBucketName`

---

### Stack 5: MonitoringStack
**Name**: `CalorieApp-{Env}-Monitoring`  
**Dependencies**: ApiStack  
**Resources**:

| Resource | Type | Configuration |
|----------|------|---------------|
| Dashboard | `aws_cloudwatch.Dashboard` | API Gateway + Lambda + DynamoDB metrics |
| ErrorAlarm | `aws_cloudwatch.Alarm` | Lambda errors > 5 in 5 min |
| LatencyAlarm | `aws_cloudwatch.Alarm` | API p95 latency > 5s |
| ThrottleAlarm | `aws_cloudwatch.Alarm` | DynamoDB throttles > 0 |
| AlarmTopic | `aws_sns.Topic` | Email notification target |

---

## IAM Roles and Permissions

### Lambda Execution Roles (per function)

| Function | Permissions |
|----------|-------------|
| AuthFunction | cognito-idp:* (on UserPool), dynamodb:GetItem/PutItem/UpdateItem (MealsNutritionTable — PROFILE# items only) |
| MealsFunction | dynamodb:* (MealsNutritionTable) |
| AiAnalysisFunction | dynamodb:PutItem (MealsNutritionTable), s3:GetObject (ImagesBucket), bedrock:InvokeModel |
| RecipesFunction | dynamodb:* (RecipesTable), dynamodb:Query (IngredientsTable — read only), bedrock:InvokeModel |
| IngredientsFunction | dynamodb:* (IngredientsTable) |
| ImagesFunction | s3:PutObject/GetObject/DeleteObject (ImagesBucket) |

**Principle of least privilege**: Each Lambda gets only the permissions it needs. No wildcard resource ARNs.

---

## Environment Configuration

| Setting | Dev | Prod |
|---------|-----|------|
| Stack prefix | `CalorieApp-Dev-` | `CalorieApp-Prod-` |
| Lambda memory | 256MB (512 for AI) | 256MB (512 for AI) |
| DynamoDB mode | On-demand | On-demand |
| S3 lifecycle | Delete after 90 days | No lifecycle (keep all) |
| CloudWatch logs retention | 7 days | 30 days |
| API throttling | 50/25 req/s | 50/25 req/s |
| PITR (DynamoDB) | Disabled | Enabled |
| S3 versioning | Disabled | Enabled |
| Alarm notifications | Optional | Required (email) |

---

## Deployment Process

```
Developer pushes code
  → Manual CDK deploy (no CI/CD pipeline for MVP)
  → cdk deploy --all --context env=dev
  → Test in dev environment
  → cdk deploy --all --context env=prod
```

**CDK Commands**:
```bash
# Deploy all stacks to dev
npx cdk deploy --all --context env=dev

# Deploy specific stack
npx cdk deploy CalorieApp-Dev-Api --context env=dev

# Diff before deploy
npx cdk diff --all --context env=dev

# Destroy (dev only)
npx cdk destroy --all --context env=dev
```
