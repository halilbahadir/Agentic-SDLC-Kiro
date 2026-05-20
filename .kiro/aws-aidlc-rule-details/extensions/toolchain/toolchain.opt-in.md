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

1. **DO NOT generate `toolchain.md` with placeholder or assumed values.** The AI MUST ask the user for their actual tool configuration before creating the file.

2. After the user selects their tools, present this follow-up prompt:

```markdown
## Toolchain Configuration Required

Please provide the actual details for your selected tools:

For Version Control:
- **Repository URL** (e.g., github.com/your-org/your-repo)
- **Default branch** (e.g., main, develop)

For Project Management (if selected):
- **Project key** (e.g., MYPROJ in Jira)
- **Board/workspace URL**

For CI/CD (if selected):
- **Pipeline configuration** (e.g., GitHub Actions, GitLab CI)

For Documentation (if selected):
- **Space/workspace URL**

[Answer]: 
```

3. Only after the user provides real configuration, create `aidlc-docs/toolchain.md` with the provided information.

4. If the user says "I'll configure later", create a minimal `toolchain.md` with `<!-- TODO: Add real configuration before using toolchain features -->` and skip toolchain sync operations until configured.

5. AI validates MCP availability for each configured tool during Workspace Detection. Tools without working MCP connections are flagged as warnings (non-blocking).
