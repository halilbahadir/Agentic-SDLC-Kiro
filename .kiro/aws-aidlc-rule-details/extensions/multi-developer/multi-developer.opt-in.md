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

When opted in:

1. **DO NOT generate `team.md` with placeholder or invented names.** The AI MUST ask the user to provide real team member details before creating the file.

2. After the user opts in, present this follow-up prompt:

```markdown
## Team Definition Required

Multi-developer mode requires real team member information. Please provide 
details for each developer who will work on this project:

For each team member, I need:
- **Alias** (username/handle)
- **Full Name**
- **Role** (e.g., Tech Lead, Sr Developer, Developer)
- **Skills** (languages, domains, specialties)

Optional governance settings:
- Is there a team lead/architect who approves design deviations? (alias)
- Should architectural deviations require approval? (yes/no)

Please list your team members:
[Answer]: 
```

3. Only after the user provides real team details, create `aidlc-docs/team.md` with the provided information.

4. If the user says "I'll fill it in later" or provides incomplete info, create `team.md` with the known details and add `<!-- TODO: Complete team member details before proceeding to Construction -->` at the top. **Block Construction phase** until all members have alias, name, role, and skills filled in.

## Toolchain Companion

The same principle applies to `toolchain.md` when the Toolchain extension is opted in — the AI must ask the user for their actual repository URL, project key, and tool configuration rather than generating placeholders.
