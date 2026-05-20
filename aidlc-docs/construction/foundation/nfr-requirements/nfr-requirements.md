# NFR Requirements — Foundation Unit

## Performance Requirements

| Metric | Target | Measurement |
|--------|--------|-------------|
| Auth operations (login, register) | < 2 seconds | End-to-end from button click to response |
| Profile CRUD | < 1 second | API response time |
| Page load (initial) | < 3 seconds | First contentful paint |
| Page navigation | < 500ms | Client-side route transitions |
| Offline sync (push) | < 5 seconds per batch | Time to push queued mutations |
| PouchDB local read | < 100ms | Local data retrieval |
| Token refresh | < 1 second | Background, non-blocking |

## Scalability Requirements

| Metric | Target | Notes |
|--------|--------|-------|
| Concurrent users | Up to 100 | Initial launch target |
| Auth requests/min | Up to 50 | Peak during login hours |
| DynamoDB read capacity | On-demand | Auto-scales, no provisioned capacity needed at this scale |
| DynamoDB write capacity | On-demand | Auto-scales |
| Lambda concurrency | Default (unreserved) | Sufficient for 100 users |
| S3 requests | Minimal | Static assets via CloudFront cache |

**Scaling Strategy**: At 100 concurrent users, AWS serverless defaults handle the load without tuning. No reserved concurrency, no provisioned DynamoDB capacity needed. Revisit if user base grows beyond 500.

## Availability Requirements

| Metric | Target | Notes |
|--------|--------|-------|
| Uptime | 99% | Acceptable for MVP/small user base |
| Planned downtime | Allowed with notice | CDK deployments may cause brief interruptions |
| Failover | None (single region) | Multi-region not needed at this scale |
| Data durability | 99.999999999% (S3/DynamoDB default) | AWS managed |
| Offline availability | Full for cached data | 30-day local cache |

**Disaster Recovery**: DynamoDB point-in-time recovery enabled. S3 versioning enabled. No cross-region replication needed.

## Reliability Requirements

| Metric | Target | Notes |
|--------|--------|-------|
| Error rate | < 1% of requests | Excluding client errors (4xx) |
| Retry policy | 3 retries, exponential backoff | For sync operations |
| Circuit breaker | Not needed | Low traffic, simple retry sufficient |
| Graceful degradation | Required | Offline mode when cloud unavailable |
| Data consistency | Eventually consistent | Server-wins for conflicts |

## Security Requirements

| Requirement | Implementation | Notes |
|-------------|---------------|-------|
| Authentication | AWS Cognito (managed) | Email/password with verification |
| Authorization | Cognito JWT at API Gateway | All protected routes require valid token |
| Data encryption at rest | DynamoDB default encryption (AES-256) | AWS managed keys |
| Data encryption in transit | HTTPS (TLS 1.2+) | CloudFront + API Gateway enforce |
| Token storage (frontend) | Access token in memory, refresh token in httpOnly cookie or secure storage | Never in localStorage |
| CORS | Restrict to app domain only | API Gateway CORS configuration |
| Rate limiting | API Gateway throttling (50 req/s burst, 25 req/s sustained) | Prevents abuse |
| Input validation | Server-side validation on all inputs | Never trust client |
| Password hashing | Cognito managed (SRP protocol) | No custom hashing needed |
| User data isolation | DynamoDB partition key = userId | Users cannot access others' data |

## Monitoring & Observability

| Component | Tool | Metrics |
|-----------|------|---------|
| Lambda functions | CloudWatch Logs | Invocations, errors, duration, throttles |
| API Gateway | CloudWatch Metrics | 4xx/5xx rates, latency (p50, p95, p99) |
| DynamoDB | CloudWatch Metrics | Read/write capacity, throttled requests |
| Cognito | CloudWatch Metrics | Sign-up/sign-in success/failure rates |
| Frontend | Browser console + error boundary | Unhandled errors, sync failures |
| Dashboards | CloudWatch Dashboard | Single dashboard with all key metrics |
| Alerts | CloudWatch Alarms → SNS | Lambda errors > 5/min, API 5xx > 1% |

## Offline Sync NFRs

| Requirement | Target | Notes |
|-------------|--------|-------|
| Local cache size | Last 30 days of data | Meals, ingredients, saved recipes, profile |
| Cache storage limit | ~50MB per user | PouchDB/IndexedDB |
| Sync latency | < 5 seconds after reconnect | For typical queue (< 20 mutations) |
| Conflict resolution | Server-wins | Automatic, no user intervention |
| Data freshness | Pull on every reconnect | Fetch latest after push completes |
| Offline write support | Manual entry, profile edits | No AI features offline |
| Cache invalidation | 24-hour TTL on stale data | Re-fetch if cache older than 24h |

## Usability NFRs

| Requirement | Target | Notes |
|-------------|--------|-------|
| Responsive design | Mobile-first, works on 320px+ | Tailwind breakpoints |
| Accessibility | WCAG 2.1 AA target | Semantic HTML, ARIA labels, keyboard nav |
| Theme support | Light + Dark mode | System preference detection |
| Loading feedback | Spinner for > 300ms operations | No blank screens |
| Error feedback | Clear error messages with retry option | No technical jargon |
| Offline indicator | Always visible when offline | Non-intrusive but clear |
