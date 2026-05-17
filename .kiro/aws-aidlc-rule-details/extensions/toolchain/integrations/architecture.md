# Architecture & Diagramming Integration

**Tools**: Miro, DrawIO, Mermaid AI  
**Category**: Architecture & Diagramming  
**Purpose**: Visual architecture diagrams, component diagrams, sequence diagrams, and collaborative whiteboarding

---

## Integration Points

| AI-DLC Stage | Operation | Direction |
|-------------|-----------|-----------|
| Application Design | Generate/export architecture diagrams | AI-DLC → Tool |
| Units Generation | Generate component/dependency diagrams | AI-DLC → Tool |
| Functional Design | Generate sequence diagrams | AI-DLC → Tool |
| Infrastructure Design | Generate infrastructure diagrams | AI-DLC → Tool |
| Any stage | Import existing diagrams for context | Tool → AI-DLC |

---

## TOOLCHAIN-ARCH-01: Diagram Generation

**When**: Design stages produce architectural artifacts  
**Operation**:

1. AI generates diagrams in Mermaid format (always — as baseline)
2. If external tool is configured AND MCP available:
   - Export/create diagram in the tool
   - Save tool URL/link in the design artifact
3. If external tool unavailable: Mermaid in markdown is the fallback

### Diagram Types by Stage

| Stage | Diagram Type | Mermaid Type |
|-------|-------------|--------------|
| Application Design | Component diagram | `flowchart` or `C4Context` |
| Application Design | System context | `C4Context` |
| Units Generation | Unit dependency graph | `flowchart` |
| Functional Design | Sequence diagrams | `sequenceDiagram` |
| Functional Design | State machines | `stateDiagram-v2` |
| NFR Design | Deployment diagram | `flowchart` |
| Infrastructure Design | Infrastructure topology | `flowchart` |

---

## TOOLCHAIN-ARCH-02: Miro Board Sync

**When**: Miro is configured and MCP available  
**Operation**:

1. Create or update Miro board for the project
2. Organize frames by AI-DLC phase:
   - Frame: "Architecture" — component diagrams
   - Frame: "Units" — unit decomposition and dependencies
   - Frame: "Sequences" — per-unit sequence diagrams
3. Export diagram content as sticky notes + connectors
4. Save board URL in design artifacts

**Note**: Miro MCP capabilities vary. If MCP only supports read, use Miro for importing existing whiteboard content into AI-DLC context.

---

## TOOLCHAIN-ARCH-03: DrawIO Integration

**When**: DrawIO is configured  
**Operation**:

1. Generate `.drawio` XML from design artifacts
2. Save to project repository (e.g., `docs/diagrams/{stage}/{diagram-name}.drawio`)
3. These files can be opened in DrawIO desktop or VS Code extension
4. Reference in design docs: `![Diagram](docs/diagrams/{path})`

**Fallback**: If no DrawIO MCP, generate Mermaid diagrams inline in markdown.

---

## TOOLCHAIN-ARCH-04: Import Existing Diagrams

**When**: Developer references existing architecture diagrams  
**Operation**:

1. Fetch diagram content via MCP (if available)
2. Parse visual elements into structured context
3. Use as input for AI-DLC design stages
4. Note source: `> Architecture imported from: [Board/File](URL)`

This enables brownfield projects to feed existing architecture into AI-DLC's reverse engineering and design stages.

---

## Mermaid as Universal Baseline

Regardless of external tool availability, AI-DLC ALWAYS generates Mermaid diagrams in markdown files. External tools are an **additional** output, not a replacement. This ensures:
- Diagrams are version-controlled with code
- No tool dependency for viewing architecture
- Diagrams render in GitHub/GitLab markdown preview
- Portable across environments
