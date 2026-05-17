# Documentation Platform Integration

**Tools**: Confluence, Notion  
**Category**: Documentation  
**Purpose**: Publish design documents, architecture decisions, and runbooks to team documentation platform

---

## Integration Points

| AI-DLC Stage | Operation | Direction |
|-------------|-----------|-----------|
| Application Design | Publish architecture overview | AI-DLC → Tool |
| Units Generation | Publish unit decomposition | AI-DLC → Tool |
| Functional Design | Publish per-unit design | AI-DLC → Tool |
| NFR Design | Publish NFR decisions | AI-DLC → Tool |
| Build & Test | Publish test strategy | AI-DLC → Tool |
| Any stage | Read existing docs for context | Tool → AI-DLC |

---

## TOOLCHAIN-DOC-01: Design Document Publishing

**When**: After a design stage is approved  
**Condition**: `auto_publish_docs: true` in toolchain.md  
**Operation**:

1. Identify approved design artifact (e.g., `application-design/`, `functional-design/`)
2. Convert markdown to tool-compatible format
3. Create or update page in documentation tool:
   - **Space/Workspace**: From `toolchain.md` Documentation Space
   - **Parent page**: Project root page (auto-created if missing)
   - **Page structure**: Mirrors aidlc-docs folder hierarchy
4. Save page URL in artifact file header
5. Log in audit.md

### Page Hierarchy (auto-created)

```
{Project Name}/
├── Architecture/
│   ├── Application Design
│   ├── Unit Decomposition
│   └── Contracts/
│       ├── {contract-1}
│       └── {contract-2}
├── Units/
│   ├── {unit-1}/
│   │   ├── Functional Design
│   │   ├── NFR Design
│   │   └── Infrastructure Design
│   └── {unit-2}/
│       └── ...
├── Testing/
│   ├── Test Strategy
│   └── Build Instructions
└── Decisions/
    └── Deviation Log
```

---

## TOOLCHAIN-DOC-02: Architecture Decision Records

**When**: Deviation is approved (multi-developer extension) or major design decision made  
**Operation**:

1. Create ADR (Architecture Decision Record) page:
   ```markdown
   # ADR-{number}: {title}
   
   **Status**: Accepted
   **Date**: {date}
   **Deciders**: {team members involved}
   
   ## Context
   {why this decision was needed}
   
   ## Decision
   {what was decided}
   
   ## Consequences
   {impact on the system}
   ```
2. Link ADR to relevant unit design pages

---

## TOOLCHAIN-DOC-03: Context Loading from Docs

**When**: AI needs additional context during any stage  
**Operation**:

1. If developer references an existing document ("see the auth design in Confluence")
2. Fetch page content via documentation MCP
3. Use as context for current stage (read-only)
4. Note source in generated artifacts: `> Source: [Page Title](URL)`

---

## TOOLCHAIN-DOC-04: Manual Publish Mode

**When**: `auto_publish_docs: false` in toolchain.md  
**Operation**:

Instead of auto-publishing, present publish prompt at stage completion:

```markdown
> 📄 **Documentation ready for publishing**
> 
> The following artifacts are ready to publish to {tool}:
> - {list of files}
>
> Publish now? (Y/N)
```

This gives the team control over when docs go live (e.g., after internal review).
