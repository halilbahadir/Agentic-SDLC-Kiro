# Multi-Developer — Opt-In

**Extension**: Multi-Developer Parallel Workflow

## Opt-In Prompt

The following question is automatically included in the Requirements Analysis clarifying questions when this extension is loaded:

```markdown
## Question: Multi-Developer Extension
Is this a multi-developer project with team-based parallel development?

A) Yes — enforce multi-developer rules: team definition, unit assignment, branching strategy, deviation gates, and integration workflow (requires `aidlc-docs/team.md` with team members defined)
B) No — single developer workflow, skip all multi-developer rules (standard AI-DLC behavior)
X) Other (please describe after [Answer]: tag below)

[Answer]: 
```

## Activation Requirements

When opted in, the following MUST exist before proceeding past Workspace Detection:
- `aidlc-docs/team.md` — team definition file with at least 2 team members

If `team.md` is missing or has fewer than 2 members, prompt the user to create it before continuing.
