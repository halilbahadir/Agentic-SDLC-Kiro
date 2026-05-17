# Multi-Developer Parallel Workflow Extension

**Extension ID**: MULTIDEV  
**Version**: 1.0.0  
**Purpose**: Enable parallel development across multiple developers with team coordination, unit assignment, branching strategy, deviation gates, and integration workflow.

---

## Rule Index

| Rule ID | Stage | Rule |
|---------|-------|------|
| MULTIDEV-01 | Workspace Detection | Load and validate team.md |
| MULTIDEV-02 | Inception (all stages) | Collaborative question answering |
| MULTIDEV-03 | Units Generation | Contract definition for cross-unit dependencies |
| MULTIDEV-04 | Units Generation | Unit assignment with AI suggestion |
| MULTIDEV-05 | Workflow Planning | Per-developer execution plan |
| MULTIDEV-06 | Construction | Branch creation per unit |
| MULTIDEV-07 | Construction | Design principle inheritance |
| MULTIDEV-08 | Construction | Deviation detection and gate |
| MULTIDEV-09 | Construction | Per-developer state tracking |
| MULTIDEV-10 | Integration | Merge order enforcement |
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

### team.md Format

```markdown
# Project Team

## Team Members

| Alias    | Name            | Role           | Skills                                   |
|----------|-----------------|----------------|------------------------------------------|
| {alias}  | {full name}     | {role}         | {comma-separated skills}                 |

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

## MULTIDEV-02: Collaborative Question Answering

**Stage**: All Inception stages  
**Enforcement**: INFORMATIONAL

### Rule

During all Inception stages (Requirements Analysis, User Stories, Application Design, Units Generation), present this guidance before questions:

```markdown
> 👥 **MULTI-DEVELOPER MODE**: Questions should be answered collaboratively 
> by the team. Involve relevant stakeholders for architectural and domain decisions.
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

## MULTIDEV-04: Unit Assignment

**Stage**: After Contract Definition (or after Units Generation if contracts skipped)  
**Enforcement**: BLOCKING — assignment must be confirmed before Construction

### Rule

1. **AI Suggests Assignment**: Analyze team skills against unit requirements:
   - Match developer skills/domains to unit technology and domain needs
   - Balance workload (story count per unit)
   - Consider dependencies (provider units to more senior developers)
   - Flag if any developer has >2 units (workload warning)

2. **Present Suggestion**:

```markdown
# 👥 Unit Assignment

## AI-Suggested Assignment

| Unit                  | Suggested Developer | Rationale                              |
|-----------------------|--------------------|----------------------------------------|
| {unit-name}           | {alias}            | {skill/domain match reasoning}         |

## Workload Summary

| Developer | Units Assigned | Total Stories |
|-----------|---------------|---------------|
| {alias}   | {count}       | {story count} |

> ⚠️ {warnings if any developer is overloaded}

## Branch Names (auto-generated)

| Unit | Branch |
|------|--------|
| {unit-name} | unit/{unit-name} |

---

**Review the assignment. Modify as needed, then confirm.**

> You may:
> 🔄 **Reassign** — swap developers between units
> ➕ **Split** — break a unit into smaller units
> ✅ **Confirm** — approve assignment and proceed
```

3. **Save Assignment**: After team confirms, save to `aidlc-docs/inception/application-design/unit-assignment.md`

4. **Calculate Merge Order**: Based on `unit-of-work-dependency.md`:
   - Units with no dependencies: Order 1 (can merge first)
   - Units depending on Order 1: Order 2
   - Continue until all units ordered
   - Units at same order level can merge in parallel

### One Developer, Multiple Units

When a developer owns multiple units:
- Each unit gets a **separate branch**
- Each unit follows a **separate Construction flow**
- Developer works them sequentially or interleaves (their choice)
- No shared state between units (full isolation)

---

## MULTIDEV-05: Per-Developer Execution Plan

**Stage**: Workflow Planning  
**Enforcement**: BLOCKING — plan must include per-developer details

### Rule

Workflow Planning MUST produce a per-developer execution plan section:

```markdown
## Per-Developer Construction Plan

### {alias} — {unit-name}
- **Branch**: unit/{unit-name}
- **Dependencies**: {list or "None"}
- **Contracts available**: {Yes/No — list which}
- **Can start**: {Immediately / After {unit} completes Functional Design}
- **Stages**: Functional Design → NFR Requirements → NFR Design → Infrastructure Design → Code Generation → Unit Tests

### {next developer...}
```

Include start conditions:
- If contract exists for dependency → "Can start: Immediately"
- If no contract and dependency exists → "Can start: After {provider-unit} completes Functional Design"

---

## MULTIDEV-06: Branch Creation

**Stage**: Construction (session start)  
**Enforcement**: BLOCKING — must be on correct branch before any Construction work

### Rule

When a developer starts their Construction session:

1. **Verify identity**: Confirm which developer is working (match against `team.md`)
2. **Create or switch to branch**: Use Git MCP to create branch from `main`:
   - Initial development: `unit/{unit-name}`
   - Bug fix: `fix/{unit-name}/{issue-id}`
   - New feature: `feat/{unit-name}/{issue-id}`
   - Hotfix (urgent): `hotfix/{issue-id}`
3. **Verify branch**: Confirm active branch matches expected pattern

### Branch Naming Convention

| Scenario | Pattern | Example |
|----------|---------|---------|
| Initial unit development | `unit/{unit-name}` | `unit/payment-service` |
| Bug fix (post-merge) | `fix/{unit-name}/{issue-id}` | `fix/payment-service/ISSUE-42` |
| New feature (post-merge) | `feat/{unit-name}/{issue-id}` | `feat/auth-service/ISSUE-55` |
| Hotfix (urgent, any unit) | `hotfix/{issue-id}` | `hotfix/ISSUE-99` |

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
- [ ] Submitted by {alias} ({date})
- [ ] Approved by {approver} ({date})
- [ ] Rejected by {approver} ({date}) — Reason: ___
```

---

## MULTIDEV-09: Per-Developer State Tracking

**Stage**: Construction (continuous)  
**Enforcement**: BLOCKING — state must be updated after each stage completion

### Rule

Each unit maintains its own state file at `aidlc-docs/construction/{unit-name}/developer-state.md`:

```markdown
# Developer State: {unit-name}

## Unit: {unit-name}
## Developer: {alias}
## Branch: unit/{unit-name}

| Stage              | Status         | Started    | Completed  |
|-------------------|----------------|------------|------------|
| Functional Design | {status}       | {date}     | {date}     |
| NFR Requirements  | {status}       | {date}     | {date}     |
| NFR Design        | {status}       | {date}     | {date}     |
| Infrastructure Design | {status}   | {date}     | {date}     |
| Code Generation   | {status}       | {date}     | {date}     |
| Unit Tests        | {status}       | {date}     | {date}     |

## Deviations
- [{status}] {unit}-{number}: {description} ({APPROVED/PENDING/REJECTED} by {alias})

## Blockers
- {description or "None"}
```

Status values: ⏳ Pending | 🔄 In Progress | ✅ Complete | ❌ Blocked

Additionally, update global `aidlc-state.md` with per-unit summary:

```markdown
## Per-Unit State

| Unit | Developer | Branch | Status | Merged |
|------|-----------|--------|--------|--------|
| {unit-name} | {alias} | unit/{unit-name} | {status} | {Yes/No} |
```

---

## MULTIDEV-10: Merge Order Enforcement

**Stage**: Integration  
**Enforcement**: BLOCKING — merge requests must follow dependency order

### Rule

Before a merge request is approved:

1. Check `unit-assignment.md` merge order
2. Verify all dependency units are already merged to main
3. If dependencies not merged:
   - If contracts exist → allow merge (contract tests validate compatibility)
   - If no contracts → block merge, notify developer: *"{dependency-unit} must merge first"*

### Exception: Contract-Based Independence

If contracts were defined in Inception for all cross-unit interfaces, units MAY merge in any order because:
- Contract tests validate interface compatibility
- Actual implementation details are abstracted behind the contract
- Integration tests (Level 2) catch any contract violations post-merge

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
| MULTIDEV-08 | N/A | No deviations detected |
| MULTIDEV-09 | ✅ Compliant | State updated |
| MULTIDEV-10 | N/A | Not at integration stage |
| MULTIDEV-11 | N/A | Not at integration stage |
| MULTIDEV-12 | N/A | Not at integration stage |
```
