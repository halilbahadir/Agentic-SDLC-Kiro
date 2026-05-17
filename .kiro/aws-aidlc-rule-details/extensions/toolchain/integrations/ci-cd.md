# CI/CD Integration

**Tools**: GitHub Actions, GitLab CI, Jenkins, Bamboo, AWS CodePipeline  
**Category**: CI/CD  
**Purpose**: Build pipeline triggers, test execution, deployment automation, status reporting

---

## Integration Points

| AI-DLC Stage | Operation | Direction |
|-------------|-----------|-----------|
| Construction (push) | Trigger pipeline | AI-DLC → Tool |
| Construction (MR) | Check pipeline status | Tool → AI-DLC |
| Build & Test | Generate pipeline config | AI-DLC → Tool |
| Integration | Trigger integration tests | AI-DLC → Tool |
| Integration | Report test results | Tool → AI-DLC |

---

## TOOLCHAIN-CI-01: Pipeline Configuration Generation

**When**: Build & Test stage  
**Operation**:

Based on project technology stack (from NFR Requirements), generate CI/CD pipeline configuration:

### GitHub Actions
```yaml
# .github/workflows/unit-ci.yml (generated)
name: Unit CI
on:
  push:
    branches: ['unit/**', 'fix/**', 'feat/**']
  pull_request:
    branches: [main]
jobs:
  build-and-test:
    # ... generated based on tech stack
```

### GitLab CI
```yaml
# .gitlab-ci.yml (generated)
stages:
  - build
  - test
  - security
# ... generated based on tech stack
```

The AI generates the appropriate config file format based on the CI/CD tool in `toolchain.md`.

---

## TOOLCHAIN-CI-02: Pipeline Trigger on Push

**When**: Developer pushes to remote  
**Condition**: `trigger_ci_on_push: true` in toolchain.md  
**Operation**:

1. Push triggers CI pipeline automatically (via VCS webhook — no MCP action needed)
2. AI monitors pipeline status via MCP (if available)
3. Reports result to developer:
   ```markdown
   ## CI Pipeline Result
   - **Pipeline**: {URL}
   - **Status**: ✅ Passed / ❌ Failed
   - **Duration**: {time}
   - **Failed step**: {step name, if failed}
   - **Logs**: {URL to logs}
   ```

---

## TOOLCHAIN-CI-03: Pre-Merge Pipeline Check

**When**: PR/MR is created  
**Operation**:

1. CI pipeline runs on PR/MR branch
2. AI checks pipeline status before allowing merge approval
3. If pipeline fails:
   - Report failure details
   - Suggest fix based on error logs (if accessible via MCP)
   - Block merge recommendation until pipeline passes

---

## TOOLCHAIN-CI-04: Integration Test Pipeline

**When**: Unit merges to main (Level 2 testing)  
**Operation**:

1. Main branch CI triggers integration test suite
2. If integration tests fail:
   - Identify failing tests and affected units
   - Notify unit owners
   - Create fix branch suggestion
3. If all pass:
   - Update global state: unit fully integrated
   - Notify team: *"{unit-name} successfully integrated"*

---

## TOOLCHAIN-CI-05: Pipeline Config per Unit (Multi-Developer)

**When**: Multi-developer extension active with multiple units  
**Operation**:

Generate pipeline config that:
- Runs unit-specific tests only for changed unit (path-based triggers)
- Runs full integration suite on main branch
- Supports parallel unit pipelines

Example (GitHub Actions with path filters):
```yaml
on:
  push:
    paths:
      - 'payment-service/**'
    branches: ['unit/payment-service']
```

This prevents unnecessary CI runs — each developer's push only triggers their unit's tests.

---

## Pipeline Status in Developer State

Update `developer-state.md` with CI information:

```markdown
## CI/CD Status
- **Last Pipeline**: {URL}
- **Status**: {passing/failing}
- **Coverage**: {percentage}%
- **Last Run**: {timestamp}
- **Blocking Issues**: {none or list}
```
