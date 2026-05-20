# Toolchain Configuration

## Project Context

| Setting | Value |
|---------|-------|
| Project Key | CALORIE-APP |
| Repository | halilbahadir/Agentic-SDLC-Kiro |
| Documentation Space | N/A (local only) |

## Active Tools

| Category | Tool | MCP Server Name | Status | Purpose |
|----------|------|-----------------|--------|---------|
| Version Control | GitHub | github-mcp | ❌ Not configured | Branch creation, PRs, code review |

> ⚠️ **Degraded Mode**: `github-mcp` is not found in `.kiro/settings/mcp.json`. 
> Git operations will use local `git` CLI instead. To enable full GitHub MCP integration,
> add a GitHub MCP server to your `.kiro/settings/mcp.json` configuration.

## Integration Preferences

| Setting | Value | Notes |
|---------|-------|-------|
| auto_sync_stories | false | Stories stay in AI-DLC docs |
| auto_create_branches | true | Create branches via Git for each unit |
| auto_publish_docs | false | Docs stay local in aidlc-docs/ |
| link_commits_to_issues | true | Include issue IDs in commit messages |
| trigger_ci_on_push | false | No CI/CD configured |
