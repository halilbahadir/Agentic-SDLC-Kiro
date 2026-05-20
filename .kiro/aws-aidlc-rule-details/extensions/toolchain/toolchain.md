# Toolchain Integration Extension

**Extension ID**: TOOLCHAIN  
**Version**: 1.0.0  
**Purpose**: Integrate AI-DLC workflow with enterprise tools (project management, version control, documentation, architecture, CI/CD) via MCP servers configured in Kiro.

---

## Rule Index

| Rule ID | Stage | Rule |
|---------|-------|------|
| TOOLCHAIN-01 | Workspace Detection | Load and validate toolchain.md |
| TOOLCHAIN-02 | Workspace Detection | MCP availability validation |
| TOOLCHAIN-03 | All stages | Bidirectional sync behavior |
| TOOLCHAIN-04 | All stages | Graceful degradation |
| TOOLCHAIN-05 | All stages | Audit trail for external operations |

---

## TOOLCHAIN-01: Load and Validate Toolchain Configuration

**Stage**: Workspace Detection  
**Enforcement**: BLOCKING (if extension opted-in)

### Rule

During Workspace Detection, load `aidlc-docs/toolchain.md` and validate:
1. File exists and contains `## Active Tools` section
2. Each tool entry has: Category, Tool, MCP Server Name, Purpose
3. MCP Server Names are non-empty strings

If `toolchain.md` does not exist, prompt user to create it using the template below.

### toolchain.md Format

```markdown
# Toolchain Configuration

## Project Context

| Setting | Value |
|---------|-------|
| Project Key | {e.g., MYPROJ — used for Jira, branch prefixes} |
| Repository | {e.g., org/repo-name — used for VCS operations} |
| Documentation Space | {e.g., TEAM — used for Confluence space key} |

## Active Tools

| Category | Tool | MCP Server Name | Purpose |
|----------|------|-----------------|---------|
| Project Management | {Jira/Linear/Azure DevOps} | {mcp-name} | {what it's used for} |
| Version Control | {GitHub/GitLab/Bitbucket} | {mcp-name} | {what it's used for} |
| Documentation | {Confluence/Notion} | {mcp-name} | {what it's used for} |
| Architecture | {Miro/DrawIO} | {mcp-name} | {what it's used for} |
| CI/CD | {GitHub Actions/GitLab CI/Jenkins} | {mcp-name} | {what it's used for} |

## Integration Preferences

| Setting | Value | Notes |
|---------|-------|-------|
| auto_sync_stories | {true/false} | Auto-create stories in project management tool |
| auto_create_branches | {true/false} | Create branches via VCS MCP (vs. local git) |
| auto_publish_docs | {true/false} | Publish design docs to documentation tool |
| link_commits_to_issues | {true/false} | Include issue IDs in commit messages |
| trigger_ci_on_push | {true/false} | Trigger CI pipeline after push |
```

### Example: Enterprise Java Project

```markdown
# Toolchain Configuration

## Project Context

| Setting | Value |
|---------|-------|
| Project Key | PAYMENTS |
| Repository | acme-corp/payment-platform |
| Documentation Space | ENGTEAM |

## Active Tools

| Category | Tool | MCP Server Name | Purpose |
|----------|------|-----------------|---------|
| Project Management | Jira | jira-mcp | Epic/story sync, sprint tracking, defect management |
| Version Control | GitHub | github-mcp | Branch creation, PRs, code review |
| Documentation | Confluence | confluence-mcp | Architecture docs, design decisions, runbooks |
| Architecture | Mermaid | (built-in) | Sequence diagrams, component diagrams |
| CI/CD | GitHub Actions | github-mcp | Build, test, deploy pipelines |

## Integration Preferences

| Setting | Value | Notes |
|---------|-------|-------|
| auto_sync_stories | true | Create Jira stories from AI-DLC user stories |
| auto_create_branches | true | Use GitHub MCP for branch operations |
| auto_publish_docs | false | Manual publish after team review |
| link_commits_to_issues | true | Format: "feat(PAYMENTS-123): description" |
| trigger_ci_on_push | true | GitHub Actions triggers on push |
```

---

## TOOLCHAIN-02: MCP Availability Validation

**Stage**: Workspace Detection  
**Enforcement**: WARNING (non-blocking — workflow continues with available tools)

### Rule

For each tool in `toolchain.md`, validate MCP connectivity:

1. **Check MCP is configured in workspace**: Read `.kiro/mcp.json` (or the workspace MCP configuration file) and verify the MCP server name from `toolchain.md` exists as a configured server. If the server name is NOT found in `mcp.json`:
   - Flag as ❌ **Not configured**
   - Inform user: *"The MCP server '{name}' referenced in toolchain.md is not configured in your workspace's `.kiro/mcp.json`. Please add it to your MCP configuration or update toolchain.md with the correct server name."*
   - Provide the expected `mcp.json` entry format for the missing server

2. **Test connection**: If MCP is configured, attempt a lightweight read operation:
   - Jira: List projects or get project info
   - GitHub/GitLab: List repositories or get repo info
   - Confluence: List spaces or get space info
   - Miro: List boards (if MCP available)

3. **Report status**:

```markdown
# 🔌 Toolchain Validation

| Tool | MCP Server | Status | Details |
|------|-----------|--------|---------|
| Jira | jira-mcp | ✅ Connected | Project "PAYMENTS" accessible |
| GitHub | github-mcp | ✅ Connected | Repo "acme-corp/payment-platform" accessible |
| Confluence | confluence-mcp | ❌ Not configured | MCP not found in Kiro config |
| Mermaid | (built-in) | ✅ Available | No MCP needed |
| GitHub Actions | github-mcp | ✅ Connected | Via GitHub MCP |

## Actions Required
- ⚠️ **Confluence**: MCP not configured. Options:
  - Install confluence-mcp in Kiro and retry
  - Remove from toolchain.md (docs will stay local only)
  - Continue without — documentation sync will be skipped
```

4. **Graceful handling**: If a tool's MCP is unavailable:
   - Mark tool as `degraded` in state
   - AI-DLC continues with local-only behavior for that category
   - Remind user at relevant stages: *"Confluence sync skipped — MCP not available"*

---

## TOOLCHAIN-03: Bidirectional Sync Behavior

**Stage**: All stages (when integration points are reached)  
**Enforcement**: CONDITIONAL (based on integration preferences in toolchain.md)

### Rule

At each AI-DLC stage, check if a toolchain integration point exists. If yes AND the tool is connected AND the preference is enabled, execute the sync.

### Integration Points by Stage

**See integration-specific rule files for detailed behavior:**
- `integrations/project-management.md` — story sync, issue creation, sprint tracking
- `integrations/version-control.md` — branch ops, PR/MR creation, code review
- `integrations/documentation.md` — doc publishing, page creation
- `integrations/architecture.md` — diagram sync, visual exports
- `integrations/ci-cd.md` — pipeline triggers, status checks

### Sync Direction

| Operation | Direction | Example |
|-----------|-----------|---------|
| Story creation | AI-DLC → Tool | Stories generated → created in Jira |
| Issue pickup | Tool → AI-DLC | Developer picks Jira issue → starts branch |
| Branch creation | AI-DLC → Tool | Unit assigned → branch created via MCP |
| PR creation | AI-DLC → Tool | Construction complete → PR opened via MCP |
| CI status | Tool → AI-DLC | Pipeline result → reported in developer-state |
| Doc publish | AI-DLC → Tool | Design approved → published to Confluence |

---

## TOOLCHAIN-04: Graceful Degradation

**Stage**: All stages  
**Enforcement**: MANDATORY

### Rule

If a configured tool becomes unavailable mid-workflow (MCP disconnects, API error, permission denied):

1. **Do NOT block the workflow** — continue with local-only behavior
2. **Log the failure** in `aidlc-docs/audit.md`:
   ```markdown
   ## Toolchain Sync Failure
   **Timestamp**: {ISO timestamp}
   **Tool**: {tool name}
   **Operation**: {what was attempted}
   **Error**: {error message}
   **Fallback**: {what happened instead — e.g., "saved locally only"}
   ```
3. **Queue for retry**: Note the failed sync for manual retry later
4. **Inform user**: *"⚠️ {Tool} sync failed: {reason}. Work saved locally. You can retry sync later."*

### Local-Only Fallback Behavior

| Category | Normal (MCP available) | Degraded (MCP unavailable) |
|----------|----------------------|---------------------------|
| Project Management | Stories synced to Jira | Stories in `stories.md` only |
| Version Control | Branch via MCP | Branch via local `git` CLI |
| Documentation | Published to Confluence | Saved in `aidlc-docs/` only |
| Architecture | Exported to Miro | Mermaid in markdown only |
| CI/CD | Pipeline triggered via MCP | Manual trigger instructions provided |

---

## TOOLCHAIN-05: Audit Trail for External Operations

**Stage**: All stages  
**Enforcement**: MANDATORY

### Rule

Every external tool operation MUST be logged in `aidlc-docs/audit.md`:

```markdown
## External Tool Operation
**Timestamp**: {ISO timestamp}
**Tool**: {tool name}
**Operation**: {create/update/read/delete}
**Target**: {what was affected — e.g., "PAYMENTS-42", "branch unit/auth-service"}
**Status**: {success/failed}
**Details**: {URL or ID of created resource}
```

This ensures traceability even if the external tool's history is lost or inaccessible.

---

## State Update

Record in `aidlc-state.md`:
```markdown
## Extension Configuration
- Toolchain: Enabled
- Tools: {comma-separated list of active tools}
- MCP Status: {N}/{total} connected
- Degraded: {list of unavailable tools, or "None"}
```

---

## Compliance Summary Template

```markdown
## Toolchain Extension Compliance

| Rule | Status | Notes |
|------|--------|-------|
| TOOLCHAIN-01 | ✅ Compliant | toolchain.md loaded, {N} tools configured |
| TOOLCHAIN-02 | ⚠️ Partial | {N-1}/{N} MCPs connected, {tool} degraded |
| TOOLCHAIN-03 | ✅ Compliant | Sync executed for {operations} |
| TOOLCHAIN-04 | ✅ Compliant | Graceful degradation active for {tool} |
| TOOLCHAIN-05 | ✅ Compliant | {N} operations logged |
```
