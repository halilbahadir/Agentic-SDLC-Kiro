# Tech Stack Decisions — Foundation Unit

## Infrastructure Organization (Multi-Stack CDK)

| Stack | Resources | Rationale |
|-------|-----------|-----------|
| **AuthStack** | Cognito User Pool, User Pool Client | Independent auth lifecycle, rarely changes |
| **StorageStack** | DynamoDB tables, S3 buckets | Data layer independent of compute |
| **ApiStack** | API Gateway, Lambda functions, IAM roles | Compute layer, changes frequently |
| **HostingStack** | CloudFront distribution, S3 (frontend), Route53 (if custom domain) | Frontend hosting, CDN config |

**Stack Dependencies:**
```
AuthStack (no deps)
StorageStack (no deps)
ApiStack → depends on AuthStack + StorageStack
HostingStack (no deps, deploys frontend independently)
```

**Benefits of multi-stack:**
- Deploy API changes without touching auth or storage
- Independent rollback per concern
- Clearer IAM boundaries
- Parallel stack deployments

---

## Tech Stack Summary

| Layer                 | Technology         | Version           | Rationale                                                    |
| --------------------- | ------------------ | ----------------- | ------------------------------------------------------------ |
| **Frontend**          | React              | 18.x              | Team expertise, large ecosystem                              |
| **Frontend Language** | TypeScript         | 5.x               | Type safety, better DX                                       |
| **Build Tool**        | Vite               | 5.x               | Fast builds, HMR, ESM-native                                 |
| **Styling**           | Tailwind CSS       | 3.x               | Utility-first, responsive, dark mode built-in                |
| **Offline Storage**   | PouchDB            | 8.x               | Built-in sync protocol, IndexedDB backend                    |
| **HTTP Client**       | axios              | 1.x               | Interceptors for auth tokens, error handling                 |
| **Routing**           | React Router       | 6.x               | Standard React routing                                       |
| **Charts**            | Recharts           | 2.x               | React-native charts, responsive                              |
| **Backend Runtime**   | Node.js            | 20.x              | LTS, Lambda-supported, team expertise                        |
| **Backend Language**  | TypeScript         | 5.x               | Shared types with frontend                                   |
| **IaC**               | AWS CDK            | 2.x (TypeScript)  | Type-safe infrastructure, same language as app               |
| **Database**          | DynamoDB           | On-demand         | Serverless, auto-scaling, no management                      |
| **Auth**              | AWS Cognito        | —                 | Managed auth, email/password, JWT tokens                     |
| **API**               | API Gateway (REST) | —                 | Managed, Cognito authorizer integration                      |
| **Storage**           | S3                 | —                 | Image storage, frontend hosting                              |
| **CDN**               | CloudFront         | —                 | Global edge caching, HTTPS                                   |
| **Monitoring**        | CloudWatch         | —                 | Logs, metrics, dashboards, alarms                            |
| **AI/ML**             | AWS Bedrock        | Claude 4.6 Sonnet | Multimodal (images), recipe generation (used by other units) |

---

## Key Technical Decisions

### 1. DynamoDB On-Demand Mode
**Decision**: Use on-demand (pay-per-request) instead of provisioned capacity.  
**Rationale**: At 100 concurrent users, traffic is unpredictable and low. On-demand eliminates capacity planning and costs nothing when idle. Switch to provisioned only if costs become significant at scale.

### 2. Lambda Cold Start Mitigation
**Decision**: Accept cold starts (no provisioned concurrency).  
**Rationale**: At 100 users with 2-second response time target, cold starts (typically 500ms-1s for Node.js) are acceptable. Provisioned concurrency adds cost with minimal benefit at this scale.

### 3. PouchDB for Offline Sync
**Decision**: Use PouchDB with custom sync logic (not CouchDB backend).  
**Rationale**: PouchDB provides IndexedDB storage and document-based local DB. Since our backend is DynamoDB (not CouchDB), we implement custom sync logic that uses PouchDB locally and REST API for server communication. Server-wins conflict resolution simplifies the sync algorithm.

### 4. Token Storage Strategy
**Decision**: Access token in memory (React state), refresh token in httpOnly cookie.  
**Rationale**: Memory storage prevents XSS access to tokens. httpOnly cookie prevents JavaScript access to refresh token. Trade-off: page refresh requires token refresh call, but this is acceptable.

### 5. Single React App (Not Micro-Frontends)
**Decision**: Single React SPA with module-based code splitting.  
**Rationale**: At this scale and team size (2 developers), micro-frontends add unnecessary complexity. Code splitting via React.lazy() provides sufficient module isolation.

### 6. Flat Monorepo Structure
**Decision**: Single package.json, flat `src/` directory structure.  
**Rationale**: Team chose simplicity over workspace tooling. At 2 developers and 3 units, the overhead of npm workspaces/turborepo isn't justified. Shared types are imported via relative paths.

### 7. Multi-Stack CDK
**Decision**: 4 separate CDK stacks (Auth, Storage, API, Hosting).  
**Rationale**: Enables independent deployment of each concern. API changes (most frequent) don't require redeploying auth or storage. Clearer blast radius for each deployment.

---

## Dependencies (npm packages)

### Frontend
```json
{
  "react": "^18.x",
  "react-dom": "^18.x",
  "react-router-dom": "^6.x",
  "axios": "^1.x",
  "pouchdb-browser": "^8.x",
  "recharts": "^2.x",
  "tailwindcss": "^3.x"
}
```

### Backend
```json
{
  "@aws-sdk/client-dynamodb": "^3.x",
  "@aws-sdk/lib-dynamodb": "^3.x",
  "@aws-sdk/client-cognito-identity-provider": "^3.x",
  "@aws-sdk/client-s3": "^3.x",
  "@aws-sdk/s3-request-presigner": "^3.x"
}
```

### Infrastructure
```json
{
  "aws-cdk-lib": "^2.x",
  "constructs": "^10.x"
}
```

### Dev Dependencies
```json
{
  "typescript": "^5.x",
  "vite": "^5.x",
  "@vitejs/plugin-react": "^4.x",
  "vitest": "^1.x",
  "@testing-library/react": "^14.x",
  "eslint": "^8.x",
  "prettier": "^3.x"
}
```
