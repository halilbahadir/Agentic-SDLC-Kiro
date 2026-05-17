# Project Management Integration

**Tools**: Jira, Linear, Azure DevOps  
**Category**: Project Management  
**Purpose**: Sync stories, track issues, manage sprints, handle defects

---

## Integration Points

| AI-DLC Stage | Operation | Direction |
|-------------|-----------|-----------|
| User Stories | Create epics and stories | AI-DLC → Tool |
| Units Generation | Create components/labels per unit | AI-DLC → Tool |
| Unit Assignment | Assign stories to developers | AI-DLC → Tool |
| Construction (start) | Move story to "In Progress" | AI-DLC → Tool |
| Construction (complete) | Move story to "Done" / "In Review" | AI-DLC → Tool |
| Defect found | Create bug ticket | AI-DLC → Tool |
| Issue pickup | Developer selects issue to work on | Tool → AI-DLC |

---

## TOOLCHAIN-PM-01: Story Sync

**When**: After User Stories stage completes and is approved  
**Condition**: `auto_sync_stories: true` in toolchain.md  
**Operation**:

1. Read approved stories from `aidlc-docs/inception/user-stories/stories.md`
2. For each story, create issue in project management tool:
   - **Type**: Story (or Epic for large groupings)
   - **Title**: Story title
   - **Description**: Story narrative + acceptance criteria
   - **Labels**: AI-DLC generated, unit name (if assigned)
   - **Project**: From `toolchain.md` Project Key
3. Save mapping in `aidlc-docs/inception/user-stories/tool-mapping.md`:

```markdown
# Story ↔ Tool Mapping

| Story ID (AI-DLC) | External ID | Tool | URL |
|-------------------|-------------|------|-----|
| US-001 | PAYMENTS-101 | Jira | https://jira.example.com/browse/PAYMENTS-101 |
| US-002 | PAYMENTS-102 | Jira | https://jira.example.com/browse/PAYMENTS-102 |
```

---

## TOOLCHAIN-PM-02: Unit Component Mapping

**When**: After Units Generation completes  
**Operation**:

1. For each unit in `unit-of-work.md`, create a component/label in the tool
2. Link stories to their unit's component (from `unit-of-work-story-map.md`)

---

## TOOLCHAIN-PM-03: Assignment Sync

**When**: After Unit Assignment is confirmed (multi-developer extension)  
**Operation**:

1. Read `unit-assignment.md`
2. For each developer-unit mapping, assign corresponding stories in the tool
3. Update tool-mapping.md with assignee info

---

## TOOLCHAIN-PM-04: Status Transitions

**When**: Developer starts/completes Construction stages  
**Operation**:

| Developer Action | Tool Status Change |
|-----------------|-------------------|
| Starts Functional Design | Story → "In Progress" |
| Completes Code Generation | Story → "In Review" |
| MR merged | Story → "Done" |
| Blocked (deviation pending) | Story → "Blocked" |

---

## TOOLCHAIN-PM-05: Defect Creation

**When**: Bug discovered during Build & Test or Integration  
**Operation**:

1. Create bug ticket in tool:
   - **Type**: Bug
   - **Title**: AI-generated from test failure
   - **Description**: Failure details, steps to reproduce, expected vs actual
   - **Component**: Unit where bug was found
   - **Priority**: Based on severity (blocking = Critical, non-blocking = Medium)
   - **Assignee**: Unit owner (from unit-assignment.md)
2. Log in audit.md
3. Developer picks up bug → creates fix branch (same as issue pickup flow)

---

## TOOLCHAIN-PM-06: Issue Pickup (Tool → AI-DLC)

**When**: Developer says "I'm picking up ISSUE-42" or "work on PAYMENTS-42"  
**Operation**:

1. Fetch issue details from tool via MCP
2. Determine unit (from component/label)
3. Create appropriate branch: `fix/{unit-name}/{issue-id}` or `feat/{unit-name}/{issue-id}`
4. Load unit context (same as Construction start)
5. Update issue status to "In Progress"

---

## Commit Message Format

When `link_commits_to_issues: true`:

```
{type}({issue-id}): {description}

Examples:
feat(PAYMENTS-101): implement payment creation endpoint
fix(PAYMENTS-142): handle null amount in refund validation
test(PAYMENTS-101): add integration tests for payment flow
```
