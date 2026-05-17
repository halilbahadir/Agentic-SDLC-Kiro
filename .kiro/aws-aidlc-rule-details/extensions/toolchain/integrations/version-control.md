# Version Control Platform Integration

**Tools**: GitHub, GitLab, Bitbucket  
**Category**: Version Control  
**Purpose**: Branch operations, pull/merge requests, code review via MCP

---

## Integration Points

| AI-DLC Stage | Operation | Direction |
|-------------|-----------|-----------|
| Construction (start) | Create branch | AI-DLC → Tool |
| Construction (complete) | Create PR/MR | AI-DLC → Tool |
| Integration | Merge PR/MR | AI-DLC → Tool (after approval) |
| Integration | Check CI status | Tool → AI-DLC |
| Any stage | Link commits to issues | AI-DLC → Tool |

---

## TOOLCHAIN-VCS-01: Branch Creation via MCP

**When**: Developer starts Construction for a unit  
**Condition**: `auto_create_branches: true` in toolchain.md  
**Operation**:

1. Determine branch name from convention:
   - Initial: `unit/{unit-name}`
   - Fix: `fix/{unit-name}/{issue-id}`
   - Feature: `feat/{unit-name}/{issue-id}`
   - Hotfix: `hotfix/{issue-id}`
2. Create branch from `main` via VCS MCP
3. Confirm branch creation to developer
4. Log in audit.md

**Fallback** (MCP unavailable): Use local `git checkout -b {branch-name}`

---

## TOOLCHAIN-VCS-02: Pull/Merge Request Creation

**When**: Developer completes Construction and requests merge  
**Operation**:

1. Verify pre-merge checks pass (unit tests, contract tests, security scan)
2. Ensure branch is rebased on latest `main`
3. Generate MR/PR description (see multi-developer extension MULTIDEV-12 for template)
4. Create PR/MR via VCS MCP:
   - **Title**: `[{unit-name}] {summary}` or `[{issue-id}] {summary}`
   - **Description**: Auto-generated from aidlc-docs
   - **Labels**: unit name, story IDs
   - **Reviewers**: Team members (from team.md, excluding author)
   - **Linked issues**: From tool-mapping.md (if project management enabled)
5. Save PR/MR URL in developer-state.md
6. Log in audit.md

---

## TOOLCHAIN-VCS-03: CI Status Check

**When**: After PR/MR is created or updated  
**Operation**:

1. Poll or receive CI pipeline status via MCP
2. Update developer-state.md:
   ```markdown
   ## CI Status
   - Pipeline: {URL}
   - Status: {passing/failing/running}
   - Last run: {timestamp}
   ```
3. If CI fails: report failure details to developer for fixing

---

## TOOLCHAIN-VCS-04: Merge Execution

**When**: PR/MR is approved and CI passes  
**Condition**: Developer or lead triggers merge  
**Operation**:

1. Verify merge order (if multi-developer extension active — MULTIDEV-10)
2. Execute merge via MCP (squash merge or merge commit per team preference)
3. Delete source branch (if team preference)
4. Update developer-state.md: `Merged: ✅ Yes`
5. Update global aidlc-state.md per-unit table
6. Trigger Level 2 integration tests (if CI/CD integration active)

---

## TOOLCHAIN-VCS-05: Commit Linking

**When**: Every commit during Construction  
**Condition**: `link_commits_to_issues: true` in toolchain.md  
**Operation**:

Ensure commit messages follow format:
```
{type}({issue-id}): {description}
```

Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`

This enables automatic linking in GitHub/GitLab/Bitbucket between commits and issues.
