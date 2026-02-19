# Copilot Instructions: Decoration Platform

## Project Context

This is a **documentation-driven greenfield architecture project** for an owned Decoration Platform that replaces third-party decoration services for garment customization. The codebase is currently in specification and architecture phase (MODE B greenfield).

**Primary goal**: Reduce dependency risk on third-party services while maintaining legacy design continuity for customer orders during a configurable retention window.

## Architecture Overview

### System Structure
- **Clean Architecture**: Strict separation of domain logic from infrastructure (ports and adapters pattern)
- **Bounded Contexts**: Decoration (core), Rendering/Export (compute), Assets (library)
- **Deployment Pattern**: Stateless Decoration API + scale-out Workers (ADR-0001)
- **Data Strategy**: Relational metadata (RDS) + object storage (S3) for blobs (ADR-0002)
- **Async Processing**: SQS queue-based jobs with idempotent submission (ADR-0003)

### Key Components
```
Decoration API      → Stateless validation/persistence + job orchestration
Render/Export Workers → Scale-out compute for preview/export generation
Style Read Adapter  → Platform sync for style versions + decoration spaces
Legacy Adapter      → Read-through cache for third-party exports (ADR-0005)
```

### Critical Data Flows
1. **Save Flow**: UI → API validates constraints → persist immutable `DecorationRevision`
2. **Preview Flow**: API → enqueue job → Worker renders overlay → artifact to S3
3. **Export Flow**: API → enqueue job → Worker validates + generates → artifact to S3 (deterministic by `design_revision_id` + `style_version_id` + `export_profile_id`, ADR-0004)
4. **Legacy Flow**: Check cache → miss calls third-party → store artifact (dual-run strategy, ADR-0005)

## Domain Model Essentials

### Core Aggregates
- **DecorationDesign**: Platform identifier, pinned `style_version_id` (ADR-0006)  
- **DecorationRevision**: Immutable snapshot with per-space elements (REQ-0004)
- **DecorationElement**: `text` or `graphic` type with transforms (position/scale)
- **Asset**: Approved artwork with availability rules
- **Artifact**: Preview overlays or production exports with traceability metadata

### Domain Services
- **ConstraintEngine**: Single authoritative validator used at both save-time and export-time to prevent drift (REQ-0003)

## Requirements Traceability

All requirements reference source specifications using inline citations:
- ``
- Primary spec: [inputs/SPEC-DECOR-0001.md](inputs/SPEC-DECOR-0001.md)
- Functional: REQ-0001 through REQ-0010
- Non-functional: NFR-0001 (Scale), NFR-0002 (Availability), NFR-0004 (Security), NFR-0005 (Retention)
- Full traceability matrix: [docs/02_Requirements.md](docs/02_Requirements.md)

## Documentation Structure

### Core Documentation
- [**01_Overview.md**](docs/01_Overview.md): System goal, context diagram, primary workflows
- [**03_Architecture.md**](docs/03_Architecture.md): Bounded contexts, containers, core flows, patterns
- [**04_Domain_Model.md**](docs/04_Domain_Model.md): Aggregates and constraint engine
- [**06_API_Contracts.md**](docs/06_API_Contracts.md): Critical E2E flow sequences with idempotency
- [**10_Implementation_Plan.md**](docs/10_Implementation_Plan.md): SLICE-0001 through SLICE-0007

### Decision Records
- [**docs/adr/**](docs/adr/): ADR-0001 (API+Workers), ADR-0003 (Async+Idempotency), ADR-0004 (Determinism), ADR-0005 (Legacy), ADR-0006 (Version Pinning)

### Tracked Assumptions & Questions
- [**00_Assumptions_and_OpenQuestions.md**](docs/00_Assumptions_and_OpenQuestions.md): ASM-0001 through ASM-0010 (TODO vs Confirmed), OQ-0001 through OQ-0009

## Project-Specific Conventions

### Idempotency Pattern
All mutation endpoints support `Idempotency-Key` header; job submission checks existing job by deterministic key before enqueueing (ADR-0003).

### Immutable Revisions
Design revisions are immutable snapshots; edits create new revisions. Export identity is deterministic: `design_revision_id` + `style_version_id` + `export_profile_id` (ADR-0004).

### Style Version Pinning
Each revision pins its `style_version_id` at save time; compatibility mapping to newer versions is explicit (ADR-0006). Prevents breaking existing designs when product configurations change.

### Constraint Validation
The same ConstraintEngine validates at both editor-time save and export-time generation to eliminate drift between editing and production (REQ-0003).

### Error Handling
- Stable `error_code` in RFC7807 ProblemDetails responses
- Retries only for transient failures; validation failures are terminal
- See [docs/06_API_Contracts.md#L89](docs/06_API_Contracts.md#L89) for error codes

### Legacy Continuity
Legacy designs use dual-run with read-through artifact cache to reduce third-party dependency; does not cover first-time export during outage (ADR-0005).

## Development Workflow

### Current Phase
**Architecture/specification only** - no implementation code yet. Focus on:
- Documenting requirements with source citations
- Creating sequence diagrams for E2E flows
- Maintaining traceability between specs, architecture, ADRs

### Documentation Standards
- **Source citations required**: Use `` for all requirements
- **Mermaid diagrams**: Use `flowchart` (context/architecture), `sequenceDiagram` (E2E flows), `erDiagram` (data model), `classDiagram` (domain model)
- **TODO tracking**: Mark unresolved assumptions/questions with ASM/OQ identifiers

### Testing Strategy (Future)
- Save/reload snapshot tests (REQ-0004)
- Constraint validation vectors reused in save and export tests
- Preview/export idempotency tests
- Legacy integration tests with cache behavior
- Golden-image tests for determinism (pending ASM-0005 resolution)

## Integration Points

### Platform Integration
- **Style/Space Sync**: Read from platform-owned Style Read Model (ASM-0003)
- **Design Identity**: Platform provides stable `design_id` for decoration revisions (ASM-0009)
- **Auth**: Token-based identity with role claims (ASM-0002, TODO)

### External Dependencies
- **Third-Party Legacy Service**: Immutable during transition; export generation only (REQ-0008)
- **Salesforce**: Source-of-truth for product configuration; out of scope for ownership change

## Outstanding Decisions

Resolve these before implementation (see [00_Assumptions_and_OpenQuestions.md](docs/00_Assumptions_and_OpenQuestions.md)):
- **ASM-0005**: Export determinism semantics (font versions, PDF metadata)
- **ASM-0008**: Retention window policy (12 months default)
- **OQ-0004**: Performance/SLO targets (preview latency, save latency)
- **OQ-0005**: Export format requirements (PDF layout, per-space outputs)
- **OQ-0007**: Preview API mode (sync vs async+cache)

## Quick Reference

### Key Files
- System overview: [docs/01_Overview.md](docs/01_Overview.md)
- Architecture decisions: [docs/adr/](docs/adr/)
- Domain aggregates: [docs/04_Domain_Model.md](docs/04_Domain_Model.md)
- API flows: [docs/06_API_Contracts.md](docs/06_API_Contracts.md)

### Patterns (PAT Register)
- PAT-0001: Ports and Adapters (Dependency Inversion)
- PAT-0002: Immutable Design Revision Snapshot
- PAT-0003: Constraint Engine as Pure Domain Service
- PAT-0004: Idempotent Job Submission + Artifact Cache
- PAT-0005: Legacy Artifact Read-Through Cache
