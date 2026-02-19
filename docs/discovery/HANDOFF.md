# Handoff

> Summary, next steps, and open questions for handoff.

---

## Overview

*Add your content here.*

---

## Generated Content (Managed)

> The sections below are managed by Discovery Agent.
> Do not edit content between the `DA:BEGIN` and `DA:END` markers.

### Summary

<!-- DA:BEGIN block:handoff.summary -->
Discovery phase has produced a comprehensive specification for the Decoration Platform. The architecture follows Clean Architecture with a stateless API and scale-out worker pattern. Core domain concepts (DecorationDesign, DecorationRevision, ConstraintEngine) are defined with clear aggregate boundaries and immutability invariants.

Key architectural decisions are recorded in ADR-0001 through ADR-0008, covering deployment topology, data strategy, async processing, export determinism, legacy continuity, and style version pinning. Requirements REQ-2000 through REQ-2004 capture derived requirements from assumptions and open questions that need resolution before implementation.

The implementation plan (SLICE-0001 through SLICE-0007 in docs/10_Implementation_Plan.md) provides a sequenced delivery path. A backlog with 6 work items (SLICE-000 through SLICE-005) has been generated for execution.

14 normative candidates were discovered in documentation that may represent undocumented requirements (see docs/discovery/candidates.md). These need human triage before implementation proceeds.
<!-- DA:END block:handoff.summary -->

### Next Steps

<!-- DA:BEGIN block:handoff.next_steps -->
1. **Triage normative candidates** (SLICE-000): Review 14 normative statements in docs/discovery/candidates.md; accept or reject each as formal requirements
2. **Resolve ASM-0008**: Confirm retention window policy -- exact duration and whether artifacts or inputs must be stored (REQ-2000)
3. **Resolve ASM-0009**: Confirm platform design identity model -- stable design_id vs per-cart/session identifiers (REQ-2001)
4. **Resolve OQ-0004**: Define performance/SLO targets for save latency, preview latency, and export latency
5. **Resolve OQ-0005**: Confirm export format requirements -- PDF layout specifics, per-space vs combined outputs
6. **Resolve OQ-0007**: Decide preview API mode -- synchronous return vs async with polling/cache
7. **Resolve ASM-0005**: Define export determinism semantics -- font version handling, PDF metadata stripping
8. **Resolve ASM-0002**: Confirm authentication token format and role claim structure
9. **Begin SLICE-001**: Implement retention window policy once ASM-0008 is confirmed
10. **Establish CI pipeline**: Configure build and test gates per correctness contract
<!-- DA:END block:handoff.next_steps -->

### Open Questions

<!-- DA:BEGIN block:handoff.open_questions -->
| ID | Question | Impact | Owner | Status |
|----|----------|--------|-------|--------|
| OQ-0004 | What are the performance/SLO targets for preview latency, save latency, and export latency? | Drives infrastructure sizing and architecture decisions (sync vs async preview) | Product/Engineering | Open |
| OQ-0005 | What are the exact export format requirements? PDF layout specifics, per-space or combined outputs? | Affects Export Worker implementation and determinism semantics | Product | Open |
| OQ-0007 | Should the Preview API be synchronous (generate-and-return) or asynchronous (enqueue + poll/cache)? | Affects API contract design and client integration | Engineering | Open |
| OQ-0009 | What are the third-party legacy export API capabilities and reliability characteristics? | Determines if pre-caching strategy is sufficient or if cutoff policy is needed (REQ-2002) | Engineering | Open |
| ASM-0002 | What is the authentication token format and role claim structure? | Affects API security implementation (NFR-0004) | Platform team | TODO |
| ASM-0005 | What are the exact export determinism semantics -- font version handling, PDF metadata? | Affects golden-image test strategy and export validation (ADR-0004) | Engineering | TODO |
| ASM-0008 | What is the retention window duration and storage semantics (artifacts vs inputs)? | Affects data lifecycle policy and storage costs (REQ-2000, NFR-0005) | Product | Partial/TODO |
| ASM-0009 | Does the platform provide stable design_id or per-cart/session identifiers? | Affects core data model design for DecorationDesign aggregate (REQ-2001) | Platform team | TODO |
<!-- DA:END block:handoff.open_questions -->

