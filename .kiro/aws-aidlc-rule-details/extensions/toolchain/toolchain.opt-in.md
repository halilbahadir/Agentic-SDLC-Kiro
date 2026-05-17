# Toolchain Integration — Opt-In

**Extension**: Toolchain Integration (Enterprise Tool Connectivity)

## Opt-In Prompt

The following question is automatically included in the Requirements Analysis clarifying questions when this extension is loaded:

```markdown
## Question: Toolchain Integration Extension
What external tools will be used in this project? (Select all that apply)

A) Project Management — Jira, Linear, Azure DevOps (sync stories, track issues, sprint planning)
B) Version Control Platform — GitHub, GitLab, Bitbucket (branches, PRs/MRs, code review via MCP)
C) Documentation — Confluence, Notion (publish design docs, architecture decisions)
D) Architecture & Diagramming — Miro, DrawIO, Mermaid AI (visual design, architecture diagrams)
E) CI/CD — GitHub Actions, GitLab CI, Jenkins, Bamboo, AWS CodePipeline (build pipelines)
F) None — all work stays within AI-DLC local files only
X) Other (please describe after [Answer]: tag below)

[Answer]: 
```

## Activation Requirements

When opted in (any option except F):
- User must create `aidlc-docs/toolchain.md` with tool configuration
- AI validates MCP availability for each configured tool during Workspace Detection
- Tools without working MCP connections are flagged as warnings (non-blocking)
