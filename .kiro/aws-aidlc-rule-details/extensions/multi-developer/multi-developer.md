# Multi-Developer Parallel Workflow Extension

**Extension ID**: MULTIDEV  
**Version**: 2.0.0  
**Purpose**: Enable parallel development across multiple developers with team coordination, unit-to-team assignment, Bolt-based iteration, story-level branching, deviation gates, and integration workflow.

---

## Alignment Notes

This extension aligns with the "AI-Driven Development Lifecycle (AI-DLC) Method Definition" paper:

- **Units are assigned to teams** (1+ developers), not individual developers. Story-level assignment happens during Bolt Planning.
- **Bolts** are the iteration unit — time-boxed cycles within a Unit's Construction. Branch lifetime is tied to Bolts.
- **Mob Construction** is recommended by the AI-DLC method definition (co-located teams). This extension supports both co-located and distributed workflows. For distributed teams, contracts (MULTIDEV-03) and deviation gates (MULTIDEV-08) serve as the asynchronous equivalent of real-time mob coordination.
- **Design technique agnostic** — the AI-DLC method definition describes a DDD flavor, but this extension works with any design technique the team chooses during Application Design.
- **Roles** (Owner, Contributor, Reviewer) in this extension are assignment patterns within the Developer role — not new organizational roles. The AI-DLC method definition's formal roles remain: Product Owner + Developers.
- **Rules are AI-recommended plans** — per AI-DLC method definition Principle 10 ("No hard-wired workflows"), these rules represent what AI recommends when multi-developer mode is active. The team can modify the plan through interactive dialogue.

---

## Rule Index

| Rule ID | Stage | Rule |
|---------|-------|------|
| MULTIDEV-01 | Workspace Detection | Load and validate team.md |
| MULTIDEV-02 | Inception (all stages) | Collaborative question answering (Mob Elaboration) |
| MULTIDEV-03 | Units Generation | Contract definition for cross-unit dependencies |
| MULTIDEV-04 | Units Generation | Unit-to-team assignment |
| MULTIDEV-05 | Construction (Bolt start) | Bolt planning — story assignment to developers |
| MULTIDEV-06 | Construction | Story-level branch creation |
| MULTIDEV-07 | Construction | Design principle inheritance |
| MULTIDEV-07b | Construction (all stages) | Team context in plan.md question files |
| MULTIDEV-08 | Construction | Deviation detection and gate |
| MULTIDEV-09 | Construction | Unit state tracking (Bolt-aware) |
| MULTIDEV-10 | Construction + Integration | Merge cadence (story→unit→develop→main) |
| MULTIDEV-11 | Integration | Two-level testing |
| MULTIDEV-12 | Integration | Merge request generation |

---

## MULTIDEV-01: Load and Validate Team Definition

**Stage**: Workspace Detection  
**Enforcement**: BLOCKING

### Rule

During Workspace Detection, load `aidlc-docs/team.md` and validate:
1. File exists and contains `## Team Members` section
2. At least 2 team members defined with alias, name, role, and skills
3. If `## Governance` section exists, validate all referenced aliases exist in team members
4. **All team member entries must contain real information provided by the user** — do NOT generate placeholder names, aliases, or skills. If the file contains placeholders (e.g., `{alias}`, `developer1`, `Jane Doe` without user confirmation), treat it as invalid and prompt the user to provide real details.

### Creating team.md

**CRITICAL**: When `team.md` does not exist and the user has opted into multi-developer mode:
- Ask the user for their team members' details (alias, name, role, skills)
- Ask about governance preferences (team lead, deviation approval)
- Only then create the file with the user-provided information
- NEVER invent or assume team member identities

### team.md Format

```markdown
# Project Team

## Team Members

| Alias    | Name            | Role           | Skills                                   |
|----------|-----------------|----------------|------------------------------------------|
| {alias}  | {full name}     | {role}         | {comma-separated skills}                 |

Roles can be: Tech Lead, Sr Developer, Developer, Product Owner, Domain Expert, Designer, QA, etc.
Skills include both technical (Java, React) and domain (Payment Operations, Healthcare) expertise.

## Skill Tags (optional — aids AI assignment suggestions)

- **Languages**: {list}
- **Domains**: {list}
- **Specialties**: {list}

## Governance (optional — if absent: no lead, equal team, no deviation gates)

| Setting                       | Value    | Notes                                     |
|-------------------------------|----------|-------------------------------------------|
| lead                          | {alias}  | Optional team lead / architect             |
| design_deviation_approval     | {bool}   | Require approval for architectural deviations |
| deviation_approver            | {alias}  | Who approves deviations                    |
| auto_approve_minor_deviations | {bool}   | Skip gate for implementation-level choices |
```

### State Update

Record in `aidlc-state.md`:
```markdown
## Extension Configuration
- Multi-Developer: Enabled
- Team Size: {N}
- Governance: {Yes/No}
- Lead: {alias or "None"}
```

---

## MULTIDEV-02: Collaborative Question Answering (Mob Elaboration)

**Stage**: All Inception stages  
**Enforcement**: INFORMATIONAL

> **AI-DLC method definition alignment**: The Inception Phase uses "Mob Elaboration" — a collaborative 
> requirements elaboration and decomposition ritual in a single room with a shared screen. 
> AI proposes, the mob (PO + Developers + stakeholders) reviews and refines.

### Rule

During all Inception stages (Requirements Analysis, User Stories, Application Design, Units Generation), present this guidance before questions:

```markdown
> 👥 **MOB ELABORATION**: This is a collaborative session. The team (Product Owner, 
> Developers, and relevant stakeholders) should answer questions together. AI proposes 
> the initial breakdown — the team reviews, refines, and approves.
```

The `[Answer]:` tag mechanism is unchanged. The team fills answers together (pair/mob style, or lead collects input and fills).

---

## MULTIDEV-03: Contract Definition

**Stage**: After Units Generation (before Unit Assignment)  
**Enforcement**: RECOMMENDED (non-blocking, but AI must present recommendation)

### Rule

After Units Generation produces `unit-of-work-dependency.md`, analyze cross-unit dependencies and present contract recommendation:

1. Parse dependency matrix from `unit-of-work-dependency.md`
2. Identify all provider→consumer relationships
3. Present recommendation to team:

```markdown
# ⚡ Cross-Unit Dependencies Detected

The following units have dependencies. Defining interface contracts NOW 
enables developers to work in parallel without waiting.

| Provider Unit    | Consumer Unit        | Interface Needed        |
|-----------------|---------------------|-------------------------|
| {provider}      | {consumer}          | {interface description} |

## Recommendation

Define contracts in: `aidlc-docs/inception/contracts/`

**If contracts are defined**: All developers can start Construction immediately.
**If contracts are NOT defined**: Dependent developers may need to wait for 
the provider unit's Functional Design to complete.

> Team decides: Define contracts now, or accept sequential dependency?

A) Define all contracts now (recommended for parallel development)
B) Define contracts for critical dependencies only
C) Skip contracts — accept that some developers may need to wait

> ⚠️ **Parallelism impact**: Without contracts, dependent developers CANNOT start 
> Construction until their provider unit completes Functional Design. For a team of 
> {N} developers, this may reduce effective parallelism to sequential work.

X) Other (please describe)

[Answer]: 
```

### Contract File Format

One file per interface in `aidlc-docs/inception/contracts/`:

```markdown
# Contract: {provider-unit}-{interface-name}

## Provider: {unit-name}
## Consumers: {unit-name-1}, {unit-name-2}

## Endpoints / Interface

### {METHOD} {path-or-function}
- Request: {schema}
- Response: {schema}
- Error: {error codes}

## Auth Mechanism
{how services authenticate to each other}

## SLA (optional)
- Latency: {target}
- Availability: {target}
```

---

## MULTIDEV-04: Unit-to-Team Assignment

**Stage**: After Contract Definition (or after Units Generation if contracts skipped)  
**Enforcement**: BLOCKING — assignment must be confirmed before Construction

> **AI-DLC method definition Alignment**: A Unit is "a cohesive, self-contained work element that can be built by a single team." 
> This rule assigns Units to teams (1+ developers), not to individual developers. Story-level assignment 
> to individual developers happens during Bolt Planning (MULTIDEV-05).

### Rule

#### team.md Role Types

`team.md` contains ALL project members, not only developers:

| Role Type | Examples | Assigned to Units? | Branch Needed? |
|-----------|---------|-------------------|----------------|
| **Developer** | Frontend Dev, Backend Dev, Fullstack Dev, Data Eng | Yes — does Construction work | Yes |
| **Product Owner** | PO, Business Analyst, Domain Expert | Yes — validates, answers domain questions | TBD (may update stories/acceptance criteria) |
| **Other** | Designer, QA, Architect | Depends on contribution | TBD |

Non-developer roles contribute domain expertise (e.g., a PO with "payment operations" skill helps the payment unit's Inception questions and validates outputs). Their branch needs will be determined by practice — if they need to update artifacts in the repo, they'll need a branch.

#### Step 1: Skill-Coverage Matching

For each unit, determine the **minimum team** by matching required skills to available developers:

1. **Identify unit skill requirements** — derived from Application Design (tech stack, frameworks, domain knowledge needed)
2. **Check if any single developer covers all skills** → if yes, assign 1 developer
3. **If not, find the minimum set of developers** whose combined skills cover the unit's requirements
4. **Assign relevant non-developer roles** — POs/domain experts whose business domain matches the unit
5. **Consider dependencies** — provider units (others depend on) benefit from more experienced developers
6. **Flag workload imbalances** — if one developer is assigned to too many units

```markdown
# 👥 Unit-to-Team Assignment

## Skill Coverage Analysis

| Unit | Skills Required | Covered By |
|------|----------------|-----------|
| order-service | Python, React, SQL, Payment Domain | be-dev (Python, SQL) + fe-dev (React) + po-payments (Payment Domain) |
| auth-service | Java, Security | security-dev (Java, Security) ✅ single dev covers all |
| notification-service | TypeScript, SQS | fe-dev-2 (TypeScript) ✅ single dev covers all |

## Proposed Teams

| Unit | Developers | Non-Dev Members | Rationale |
|------|-----------|-----------------|-----------|
| order-service | be-dev, fe-dev | po-payments | No single dev has Python+React; PO has payment domain expertise |
| auth-service | security-dev | — | Single dev covers Java+Security |
| notification-service | fe-dev-2 | — | Single dev covers TypeScript+SQS |
```

#### Step 2: Responsibility Matrix

For each unit with multiple team members, generate a responsibility matrix based on `team.md` roles and skills. This matrix is **advisory** — the unit lead may adjust it at any time.

**Responsibility assignment logic:**

1. **Full-stack developers** → `Story development (end-to-end)` — assigned complete user stories including frontend, backend, and data layers
2. **Specialized developers** → scoped responsibility:
   - Frontend Dev → `Story development (frontend)`
   - Backend Dev → `Story development (backend)`
   - Data Engineer → `Story development (data)`
   - UI Designer → `Story development (UI/UX)`
3. **Product Owner / Domain Expert** → `Story ownership` — validates acceptance criteria, answers domain questions, signs off on stories
4. **Tech Lead / Sr Developer** (if also lead) → `Technical review` — reviews PRs, approves design decisions
5. **Operations roles** (DevOps, SRE, Platform Eng) → `Operations & deployment` — CI/CD, infrastructure, monitoring, deployment pipelines
6. **QA** → `QA validation` — acceptance tests, regression testing
7. **Cross-cutting roles** (if skills span multiple units) → `Integration` — cross-unit contract implementation

**Workload balancing:** When multiple developers share the same responsibility type (e.g., two backend devs), distribute stories evenly so Bolt durations remain similar across team members. Flag imbalances where one developer would take significantly longer than others.

```markdown
## Unit Responsibilities: {unit-name}

| Member | Role | Responsibility | Scope | Est. Capacity |
|--------|------|---------------|-------|---------------|
| {alias} | Fullstack Dev | Story development (end-to-end) | All layers for assigned stories | {N stories/bolt} |
| {alias} | Backend Dev | Story development (backend) | API, business logic, data access | {N stories/bolt} |
| {alias} | Frontend Dev | Story development (frontend) | UI components, client integration | {N stories/bolt} |
| {alias} | Product Owner | Story ownership | Acceptance criteria, domain decisions | All stories |
| {alias} | DevOps Eng | Operations & deployment | CI/CD, infra, monitoring | Cross-cutting |

> ℹ️ AI-recommended based on team.md roles and skills. Lead ({lead-alias}) may reassign.
> ⚖️ Workload balanced: developers have similar estimated story counts per Bolt.
```

**Workload balance check:**

```markdown
## Workload Balance

| Developer | Responsibility | Est. Stories | Est. Effort |
|-----------|---------------|-------------|-------------|
| {alias} | Story development (end-to-end) | {N} | {relative} |
| {alias} | Story development (backend) | {N} | {relative} |

> ⚠️ Imbalance detected: {alias} has {X} more stories than {alias}. 
> Consider redistributing or splitting stories.
```

If all developers are balanced (within ~20% effort), no warning is shown.

#### Step 3: Determine Story Assignment Strategy

Based on team composition within each unit, recommend how stories will be assigned to individual developers during Bolt Planning:

```markdown
# 👥 Unit-to-Team Assignment

## Assignment

| Unit | Team Members | Rationale |
|------|-------------|-----------|
| {unit-name} | {alias1} (lead), {alias2}, {alias3} | {skill match reasoning} |

## Story Assignment Strategy

Based on your team composition, stories within each unit can be assigned to developers using:

**A) By story** (best for fullstack teams)
Each developer picks complete user stories. One branch per story.
```
feature/US-123 → dev-A (full story)
feature/US-124 → dev-B (full story)
```

**B) By layer within story** (best for specialized teams: FE/BE/data)
Each story gets sub-branches by layer. Multiple devs work on one story.
```
feature/US-123/frontend → fe-dev
feature/US-123/backend → be-dev
```

**C) Custom** (team defines their own)
You define your own branch naming and assignment rules.

---

> **AI Recommendation**: {A or B} — {reason based on team skills}

Choose a strategy (A/B/C):
[Answer]: 
```

#### Step 4: If Custom (Strategy C) — Collect Team's Convention

```markdown
## Custom Branching Configuration

### Branch Naming Convention
Describe your pattern (use `{unit}`, `{story-id}`, `{developer}`, `{layer}` as placeholders):
[Answer]: 

### Merge Strategy
How do branches merge? (e.g., "feature → develop → main", "story → unit → develop → main")
[Answer]: 
```

#### Step 5: Present and Confirm

```markdown
## Workload Summary

| Developer | Units (as team member) | Estimated Stories |
|-----------|----------------------|-------------------|
| {alias}   | {unit list}          | {count}           |

---

**Review the assignment. Modify as needed, then confirm.**

> You may:
> 🔄 **Reassign** — move developers between unit teams
> ➕ **Split** — break a unit into smaller units (generates more units, not sub-packages)
> 🔀 **Change strategy** — pick a different story assignment approach
> ✅ **Confirm** — approve and proceed
```

#### Step 6: Save Assignment

Save to `aidlc-docs/inception/application-design/unit-assignment.md`:

```markdown
# Unit Assignment

## Story Assignment Strategy: {A/B/C}
## Custom Convention: {if C, the team's rules}

## Unit Teams

| Unit | Team Members | Lead |
|------|-------------|------|
| {unit-name} | {alias1}, {alias2} | {alias1} |

## Responsibility Matrix

### {unit-name}

| Member | Role | Responsibility | Scope |
|--------|------|---------------|-------|
| {alias} | {role} | {responsibility type} | {scope description} |

## Merge Order (dependency-based)

| Order | Unit | Depends On | Can merge after |
|-------|------|-----------|-----------------|
| 1 | {unit} | — | Immediately |
| 2 | {unit} | {dep-unit} | {dep-unit} merged |

## Branch Structure

main (production)
 └── develop (integration)
      └── unit/{unit-name} (unit integration branch)
           └── feature/{story-id} (story branch — one Bolt lifetime)
                ├── feature/{story-id}/{layer} (optional: layer sub-branch)
```

### Key Principle: Units Are Not Split

Units remain cohesive, indivisible work elements. If a unit is too large for one developer, the correct action is to **generate more fine-grained units** during Units Generation — not to split an existing unit into sub-packages. This aligns with the AI-DLC principle that units are "loosely coupled, enabling autonomous development."

---

## MULTIDEV-05: Bolt Planning

**Stage**: Construction (at the start of each Bolt)  
**Enforcement**: BLOCKING — stories must be assigned before developers start coding

> **AI-DLC method definition Alignment**: "A Bolt is the smallest iteration in AI-DLC, designed for the rapid 
> implementation of a Unit or a set of tasks within a Unit. Bolts emphasize intense focus 
> and high-velocity delivery, with build-validation cycles measured in hours or days."

### Rule

At the start of each Bolt, the unit team plans which stories to tackle:

#### Step 1: Load Responsibility Matrix

Load the responsibility matrix from `aidlc-docs/inception/application-design/unit-assignment.md` for this unit. Use it to pre-assign stories:

- **Story development (end-to-end)** members → assign complete stories (all layers)
- **Story development (frontend/backend/data)** members → assign story layers matching their scope
- **Story ownership** members → listed as validator on all stories in the Bolt
- **Operations & deployment** members → assigned infrastructure/deployment stories (if any)
- **QA validation** members → assigned test stories or validation tasks

**Workload balancing**: Distribute stories so that all developers finish at approximately the same time within the Bolt. Consider story complexity (from user story points or AI estimate) when balancing — not just count.

#### Step 2: AI Suggests Bolt Scope

```markdown
# ⚡ Bolt {N} Planning — {unit-name}

## Bolt Duration: {hours/days — team decides}

## Suggested Stories for This Bolt (based on responsibility matrix)

| Story ID | Title | Assigned To | Responsibility | Branch | Est. Effort |
|----------|-------|-------------|---------------|--------|-------------|
| US-{id} | {title} | {alias} | Story dev (end-to-end) | feature/US-{id} | {S/M/L} |
| US-{id} | {title} | {alias} | Story dev (backend) | feature/US-{id} | {S/M/L} |
| US-{id} | {title} | {alias} | Story dev (frontend) | feature/US-{id} | {S/M/L} |

## Story Ownership (validation)
| Story ID | Validator | Role |
|----------|-----------|------|
| US-{id} | {po-alias} | Story ownership — acceptance sign-off |

## Workload Balance

| Developer | Stories | Est. Total Effort | Status |
|-----------|---------|-------------------|--------|
| {alias} | {N} | {S+M = X points} | ⚖️ Balanced |
| {alias} | {N} | {S+M = X points} | ⚖️ Balanced |

> ⚖️ All developers estimated to complete within similar timeframe.
> (or) ⚠️ Imbalance: {alias} has ~{X}% more effort. Consider moving US-{id} to {other-alias}.

## Dependencies Within This Bolt
- US-{id} depends on US-{id} (suggest: start {dep} first)

## Bolt Exit Criteria
- All story branches merged to `unit/{unit-name}`
- Unit tests pass on unit integration branch
- Story ownership sign-off received from {po-alias}
- No unresolved deviations
```

#### Step 3: Team Confirms or Adjusts

Team may reassign stories, reduce scope, or extend the Bolt. Lead may override AI-recommended assignments from the responsibility matrix.

#### Step 4: Developers Start Work

Each developer creates their story branch and begins Construction stages for their assigned stories.

### Bolt Lifecycle

```
Bolt Start
 → Stories assigned to developers
 → Developers create feature/{story-id} branches from unit/{unit-name}
 → Construction stages per story (Design → Code → Test)
 → Stories merge back to unit/{unit-name}
Bolt End
 → All story branches merged
 → Unit integration tests pass
 → Next Bolt begins (or Unit complete)
```

### One Developer, Multiple Stories

Within a Bolt, a developer may own multiple stories:
- Each story gets its own branch
- Developer works them sequentially or interleaves
- All must merge before Bolt ends

---

## MULTIDEV-06: Branch Creation

**Stage**: Construction (session start)  
**Enforcement**: BLOCKING — must be on correct branch before any Construction work

### Rule

When a developer starts work on a story:

1. **Verify identity**: Confirm which developer is working (match against `team.md`)
2. **Verify unit team membership**: Developer must be assigned to this unit's team
3. **Create story branch**: From the unit integration branch (`unit/{unit-name}`), create the story branch
4. **Verify branch**: Confirm active branch matches expected pattern

### Default Branch Structure

```
main
 └── develop
      └── unit/{unit-name}                    ← unit integration branch (lives for unit lifetime)
           └── feature/{story-id}             ← story branch (lives for one Bolt)
                ├── feature/{story-id}/{layer} ← optional layer sub-branch
                └── ...
```

### Branch Naming by Strategy

#### Strategy A: By story (fullstack teams)

| Scenario | Pattern | Example |
|----------|---------|---------|
| Story implementation | `feature/{story-id}` | `feature/US-123` |
| Bug fix | `fix/{story-id}` | `fix/US-123` |
| Hotfix | `hotfix/{issue-id}` | `hotfix/ISSUE-99` |

#### Strategy B: By layer within story (specialized teams)

| Scenario | Pattern | Example |
|----------|---------|---------|
| Story + layer | `feature/{story-id}/{layer}` | `feature/US-123/frontend` |
| Story + layer | `feature/{story-id}/{layer}` | `feature/US-123/backend` |
| Bug fix + layer | `fix/{story-id}/{layer}` | `fix/US-123/backend` |

Layer sub-branches merge into the story branch (`feature/{story-id}`), which then merges into the unit branch.

#### Strategy C: Custom

Use the convention defined by the team in `unit-assignment.md` under `## Custom Convention`.

### Merge Direction

```
feature/{story-id}/{layer} → feature/{story-id}  (layer merges to story)
feature/{story-id} → unit/{unit-name}            (story merges to unit at Bolt end)
unit/{unit-name} → develop                       (unit merges when complete)
develop → main                                   (release)
```

---

## MULTIDEV-07: Design Principle Inheritance

**Stage**: Construction — Functional Design  
**Enforcement**: BLOCKING — principles must be loaded before design questions

### Rule

At the start of Functional Design for any unit:

1. **Load inherited principles** from `aidlc-docs/inception/application-design/` (READ-ONLY):
   - Architecture patterns
   - API style decisions
   - Auth/security patterns
   - Data access patterns
   - Error handling standards
   - Logging/observability standards

2. **Load contracts** from `aidlc-docs/inception/contracts/` (READ-ONLY):
   - Interfaces this unit provides
   - Interfaces this unit consumes

3. **Load unit context** from `aidlc-docs/inception/application-design/unit-of-work-story-map.md`:
   - Stories assigned to this unit

4. **Present to developer**:

```markdown
# 📐 Inherited Design Principles

The following principles were established during Application Design 
and apply to your unit. Your Functional Design should follow these 
unless you propose a deviation (see deviation process below).

## Architecture Patterns
{loaded from application-design}

## Contracts You Provide
{from contracts/ — interfaces other units expect from you}

## Contracts You Consume
{from contracts/ — interfaces you depend on from other units}

---

Proceeding with Functional Design questions for: **{unit-name}**
```

---

## MULTIDEV-07b: Team Context in Construction Plans

**Stage**: Construction — All unit-specific stages (Functional Design, NFR Requirements, NFR Design, Infrastructure Design, Code Generation)  
**Enforcement**: BLOCKING — team context must be included in every `*-plan.md` question file

### Rule

When generating any `*-plan.md` question file for a unit (`{unit-name}-functional-design-plan.md`, `{unit-name}-nfr-requirements-plan.md`, `{unit-name}-nfr-design-plan.md`, `{unit-name}-infrastructure-design-plan.md`, `{unit-name}-code-generation-plan.md`):

1. **Load unit team** from `aidlc-docs/inception/application-design/unit-assignment.md` — get team members and their responsibilities for this unit
2. **Inject team header** at the top of the plan file (before questions)
3. **Add collaboration reminder** addressed to the unit lead

### Team Header Format

Insert this block at the top of every `*-plan.md` file, immediately after the plan title:

```markdown
---

## 👥 Unit Team

| Member | Role | Responsibility |
|--------|------|---------------|
| {alias} ⭐ | {role} | {responsibility} (Lead) |
| {alias} | {role} | {responsibility} |
| {alias} | {role} | {responsibility} |

> 📣 **{lead-alias}**: This is a collaborative session. Please invite all team members 
> listed above to answer these questions together. Each member brings expertise relevant 
> to their responsibility — collective input produces better designs.

---
```

The ⭐ marks the unit lead. Responsibilities come from the responsibility matrix in `unit-assignment.md`.

### Why This Matters

- Team members may work in **separate AI sessions** (each developer in their own IDE). The plan file is the shared artifact they all see.
- Including team names and roles in the file itself ensures anyone opening it knows who should contribute.
- The reminder to the lead makes collaboration an explicit action, not an assumption.

---

## MULTIDEV-08: Deviation Detection and Gate

**Stage**: Construction — Functional Design (continuous during question answering)  
**Enforcement**: BLOCKING if governance.design_deviation_approval = true

### Rule

During Functional Design, continuously check developer answers against inherited principles:

1. **Detection**: After each answer, compare against loaded principles. Flag if answer contradicts an established pattern.

2. **Classification**:
   - **ARCHITECTURAL** (communication pattern, deployment model, data strategy) → gate applies
   - **PATTERN** (different auth mechanism, different error format for this unit) → gate applies
   - **IMPLEMENTATION** (library choice, algorithm selection) → auto-approve if `auto_approve_minor_deviations: true`
   - **CODE-LEVEL** (variable naming, internal structure) → never gated

3. **Gate Flow** (when applicable):

```markdown
# ⚠️ Design Deviation Detected

**Inherited Principle**: {principle from Application Design}
**Your Proposal**: {what the developer's answer implies}
**Unit**: {unit-name}
**Developer**: {alias}
**Classification**: {ARCHITECTURAL / PATTERN / IMPLEMENTATION}

---

## Rationale (required)
Why does this unit need a different approach?
[Answer]: 

## Impact on Other Units
Does this affect units that consume or provide interfaces to yours?
[Answer]: 

## Proposed Contract Update (if applicable)
If this changes an interface, what's the new contract?
[Answer]: 
```

4. **After developer provides rationale**:
   - If `design_deviation_approval: true` → save to `aidlc-docs/construction/deviation-requests/{unit-name}-{number}.md`
   - Notify developer: *"Deviation logged. Notify {deviation_approver} for approval. You may continue other work while awaiting approval."*
   - If `design_deviation_approval: false` → log deviation for visibility, developer proceeds immediately

5. **Non-blocking**: Developer can continue working on other aspects of their unit. Only the specific conflicting decision is paused until approved.

### Deviation Request File Format

```markdown
# Deviation Request: {unit-name}-{number}

## Unit: {unit-name}
## Developer: {alias}
## Date: {ISO date}
## Classification: {ARCHITECTURAL / PATTERN / IMPLEMENTATION}

## Inherited Principle
{exact principle text}

## Proposed Deviation
{developer's proposal}

## Rationale
{developer's explanation}

## Impact on Other Units
{developer's assessment}

## Proposed Contract Update
{if applicable}

## Status
- **State**: PENDING | APPROVED | REJECTED
- **Submitted**: {alias} ({date})
- **Decided**: {approver} ({date}) — {reason if rejected}
```

---

## MULTIDEV-09: Unit State Tracking

**Stage**: Construction (continuous)  
**Enforcement**: BLOCKING — state must be updated after each Bolt and stage completion

### Rule

Each unit maintains its own state file at `aidlc-docs/construction/{unit-name}/unit-state.md`:

```markdown
# Unit State: {unit-name}

## Team: {alias1}, {alias2}
## Branch: unit/{unit-name}
## Strategy: {A/B/C}

## Bolt Progress

| Bolt | Stories | Status | Started | Completed |
|------|---------|--------|---------|-----------|
| Bolt 1 | US-{id}, US-{id} | {status} | {date} | {date} |
| Bolt 2 | US-{id}, US-{id} | {status} | {date} | {date} |

## Current Bolt: {N}

| Story | Assigned To | Branch | Stage | Status |
|-------|-------------|--------|-------|--------|
| US-{id} | {alias} | feature/US-{id} | {stage} | {status} |
| US-{id} | {alias} | feature/US-{id}/{layer} | {stage} | {status} |

## Deviations
- [{status}] {unit}-{number}: {description} (APPROVED/PENDING/REJECTED by {alias})

## Blockers
- {description or "None"}
```

Status values: ⏳ Pending | 🔄 In Progress | ✅ Complete | ❌ Blocked

### Session Recovery

On session resume (crash, timeout, or new session for the same unit):
1. Read `unit-state.md` to determine current Bolt and story progress
2. Verify actual progress matches recorded state (check if artifacts exist on disk)
3. If mismatch: update state to reflect reality, then resume from the correct point
4. If state file is missing: scan `aidlc-docs/construction/{unit-name}/` for artifacts and reconstruct state

### Global State Update

Update `aidlc-state.md` with per-unit summary:

```markdown
## Per-Unit State

| Unit | Team | Branch | Current Bolt | Stories Done | Status | Merged |
|------|------|--------|-------------|-------------|--------|--------|
| {unit-name} | {aliases} | unit/{unit-name} | Bolt {N} | {done}/{total} | {status} | {Yes/No} |
```

---

## MULTIDEV-10: Merge Cadence

**Stage**: Construction + Integration  
**Enforcement**: BLOCKING — merges must follow the defined cadence

### Rule

Merge cadence is tied to Bolts and unit completion:

#### Level 1: Story → Unit (at Bolt end)

When a Bolt ends:
1. All story branches (`feature/{story-id}`) merge into the unit branch (`unit/{unit-name}`)
2. If using layer sub-branches: layer branches merge into story branch FIRST, then story merges to unit
3. Unit integration tests run on the unit branch
4. If tests fail: fix on a `fix/{story-id}` branch, merge to unit branch before next Bolt starts

```
feature/{story-id}/{layer} → feature/{story-id} → unit/{unit-name}
```

#### Level 2: Unit → Develop (at unit completion)

When all stories in a unit are complete (all Bolts done):
1. Check `unit-assignment.md` merge order
2. Verify dependency units are already merged to `develop`
3. If dependencies not merged:
   - If contracts exist → allow merge (contract tests validate compatibility)
   - If no contracts → block merge: *"{dependency-unit} must merge first"*
4. Create MR/PR: `unit/{unit-name}` → `develop`
5. Level 2 tests run (integration + E2E)

```
unit/{unit-name} → develop
```

#### Level 3: Develop → Main (release)

When all units are merged to `develop` and integration tests pass:
1. Team decides release readiness
2. Create MR/PR: `develop` → `main`
3. Full regression + performance tests run

```
develop → main
```

### Merge Order for Unit → Develop

Provider units merge first, consumer units after:
- Units with no dependencies: merge first
- Units depending on merged units: merge next
- Continue until all units merged

> **Flexibility**: When contracts exist for all cross-unit interfaces, units MAY merge in any 
> order because contract tests validate compatibility. The merge order is RECOMMENDED 
> (safest path) but not strictly enforced when contracts provide the safety net.

### Bolt-End Merge Checklist

Before merging story branches at Bolt end:
- [ ] All assigned stories for this Bolt are complete
- [ ] Story branches pass their unit tests
- [ ] No unresolved deviations blocking this story
- [ ] Layer sub-branches (if any) already merged to story branch
- [ ] Rebased on latest unit branch

---

## MULTIDEV-11: Two-Level Testing

**Stage**: Integration  
**Enforcement**: BLOCKING — both levels must pass for release readiness

### Rule

#### Level 1: Per-Unit (runs in branch CI, before merge)

| Test Type | Source | Must Pass for MR |
|-----------|--------|-----------------|
| Unit tests | Generated during Code Generation | ✅ Yes |
| Contract tests | Generated from `contracts/` definitions | ✅ Yes |
| Security scan | SAST + dependency check | ✅ Yes |
| Code coverage | Threshold from NFR Requirements | ✅ Yes |
| Linting | Project standards | ✅ Yes |

#### Level 2: Integration (runs on main after merge)

| Test Type | Source | Triggered by |
|-----------|--------|-------------|
| Integration tests | Team-written (collaborative session) | Any merge to main |
| E2E tests | Team-written (collaborative session) | Any merge to main |
| Performance tests | From NFR Design (if applicable) | Nightly or on-demand |
| Contract regression | All contracts re-validated | Any merge to main |

#### Integration Test Ownership

| Test Type | Who Writes | When |
|-----------|-----------|------|
| Unit tests | Developer (during Construction) | Per-unit Construction |
| Contract tests | AI generates from `contracts/` | After contracts defined |
| Integration tests | Team (collaborative session) | After first provider unit merges |
| E2E tests | Team (collaborative session) | After core units merged |

---

## MULTIDEV-12: Merge Request Generation

**Stage**: Integration (when developer completes Construction)  
**Enforcement**: RECOMMENDED — AI generates MR description, developer may modify

### Rule

When developer says "create merge request" or completes all Construction stages:

1. **Pre-merge checks**:
   - All unit tests pass
   - All contract tests pass
   - No unresolved deviations (if gate enabled)
   - Branch rebased on latest main
   - Developer state shows all stages ✅ Complete

2. **Generate MR description**:

```markdown
## Unit: {unit-name}
## Developer: {alias}
## Branch: unit/{unit-name} → main

### Summary
{AI-generated from functional design and code generation docs}

### Stories Implemented
- [x] {story-id}: {story title}
{for each story in unit-of-work-story-map assigned to this unit}

### Design Decisions
- Follows {principle} (per Application Design)
{list inherited principles followed}
- ⚠️ Deviation approved: {description} (deviation-{number})
{list any approved deviations}

### Contracts Fulfilled
- **Provides**: {contract-name} ({key endpoints/interfaces})
- **Consumes**: {contract-name}

### Test Results
- Unit tests: {pass}/{total} passing
- Contract tests: {pass}/{total} passing
- Coverage: {percentage}%

### Checklist
- [ ] Unit tests pass
- [ ] Contract tests pass
- [ ] No unresolved deviations
- [ ] Security scan clean
- [ ] Rebased on latest main
- [ ] Documentation updated
```

3. **Create MR** via Git MCP (GitHub PR / GitLab MR)

4. **Notify team**: *"{alias} has opened MR for {unit-name}. Review requested."*

---

## Integration Failure Handling

When Level 2 tests fail after merge:

1. **Identify scope**: Which units are involved in the failure?
2. **Classify cause**:
   - **Contract mismatch** → both provider and consumer units need fixes
   - **Implementation bug** → single unit owner fixes
3. **Create fix branch**: `fix/{unit-name}/integration-{number}`
4. **Same Construction flow**: diagnose → fix → test → MR
5. **Expedited review**: integration fixes get priority review

---

## Compliance Summary Template

At each stage completion, include:

```markdown
## Multi-Developer Extension Compliance

| Rule | Status | Notes |
|------|--------|-------|
| MULTIDEV-01 | ✅ Compliant | team.md loaded, {N} members |
| MULTIDEV-02 | ✅ Compliant | Collaborative guidance shown |
| MULTIDEV-03 | ✅ Compliant | {N} contracts defined |
| MULTIDEV-04 | ✅ Compliant | Assignment confirmed |
| MULTIDEV-05 | ✅ Compliant | Per-dev plan generated |
| MULTIDEV-06 | ✅ Compliant | Branch: unit/{name} |
| MULTIDEV-07 | ✅ Compliant | Principles loaded |
| MULTIDEV-07b | ✅ Compliant | Team context in plan file |
| MULTIDEV-08 | N/A | No deviations detected |
| MULTIDEV-09 | ✅ Compliant | State updated |
| MULTIDEV-10 | N/A | Not at integration stage |
| MULTIDEV-11 | N/A | Not at integration stage |
| MULTIDEV-12 | N/A | Not at integration stage |
```
