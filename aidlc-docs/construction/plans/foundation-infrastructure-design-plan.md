# Foundation Unit — Infrastructure Design Plan

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

## Unit Context

Infrastructure decisions already established:
- AWS cloud provider
- Multi-stack CDK (AuthStack, StorageStack, ApiStack, HostingStack)
- DynamoDB on-demand, S3, Cognito, API Gateway, Lambda (Node.js 20), CloudFront
- Standard CloudWatch monitoring

---

## Clarifying Questions

### Question 1: AWS Region
Which AWS region should the infrastructure be deployed to?

A) eu-west-1 (Ireland) — closest to UK/Europe
B) eu-west-2 (London) — UK-based
C) us-east-1 (N. Virginia) — most services available, lowest latency for Bedrock
D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

### Question 2: Environment Strategy
How many deployment environments do you need?

A) Single environment (dev/prod combined) — simplest, cheapest, suitable for MVP
B) Two environments (dev + prod) — separate stacks with different configs
C) Three environments (dev + staging + prod) — full pipeline
D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

### Question 3: Custom Domain
Will you use a custom domain for the application?

A) No — use default CloudFront and API Gateway URLs for now
B) Yes — I have a domain ready (provide domain name after [Answer]: tag)
C) Later — plan to add custom domain after MVP is working
D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Generation Steps

After questions are answered, the following steps will be executed:

- [x] Step 1: Define CDK stack structure and resource mapping
- [x] Step 2: Define deployment architecture (environments, regions)
- [x] Step 3: Define IAM roles and permissions
- [x] Step 4: Define networking and security groups
- [x] Step 5: Document infrastructure configuration
- [x] Step 6: Validate infrastructure completeness

---
