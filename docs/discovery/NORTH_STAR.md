# North Star

> Vision, goals, and success metrics for the discovery process.

---

## Overview

*Add your content here.*

---

## Generated Content (Managed)

> The sections below are managed by Discovery Agent.
> Do not edit content between the `DA:BEGIN` and `DA:END` markers.

### Summary

<!-- DA:BEGIN block:north_star.summary -->
The Decoration Platform is an owned service that replaces reliance on a third-party decoration provider for garment customization. The platform must handle the full lifecycle of decoration designs -- creation, constraint validation, preview rendering, and production export generation -- while maintaining backward compatibility with legacy designs during a configurable retention window. The core value proposition is eliminating single-vendor dependency risk for a business-critical ordering workflow, without disrupting existing customer orders that depend on the legacy service.

**Key drivers:**

- Reduce operational risk from third-party service dependency
- Maintain legacy design orderability during transition period
- Enable independent scaling and evolution of decoration capabilities
- Ensure deterministic, reproducible export outputs for production use
<!-- DA:END block:north_star.summary -->

### Goals

<!-- DA:BEGIN block:north_star.goals -->
| ID | Goal | Source |
|----|------|--------|
| G-01 | All new decoration designs are created, validated, and exported entirely on the owned platform with zero third-party calls | REQ-0001, NFR-0002 |
| G-02 | Legacy designs remain orderable for the full retention window via read-through artifact cache backed by the third-party service | REQ-0008, ADR-0005 |
| G-03 | Decoration revisions are immutable snapshots; edits produce new revisions | REQ-0004, ADR-0004 |
| G-04 | The same ConstraintEngine validates at both save-time and export-time to prevent drift | REQ-0003 |
| G-05 | Preview and export workloads scale horizontally without blocking interactive editing | NFR-0001, ADR-0001 |
| G-06 | Exports are deterministic: identical inputs produce identical outputs, keyed by design_revision_id + style_version_id + export_profile_id | ADR-0004 |
| G-07 | Style version pinning ensures saved designs remain compatible when product configurations change | ADR-0006, REQ-2004 |
<!-- DA:END block:north_star.goals -->

### Non Goals

<!-- DA:BEGIN block:north_star.non_goals -->
| ID | Non-Goal | Rationale |
|----|----------|----------|
| NG-01 | Migrating existing legacy designs to the new platform data model | Legacy designs use read-through cache (ADR-0005); migration is not required for orderability |
| NG-02 | Owning product configuration or style definitions | Salesforce remains the source-of-truth for product configuration; the platform syncs via the Style Read Model (ASM-0003) |
| NG-03 | Building a general-purpose graphic design editor | The platform handles decoration-specific elements (text, graphics with transforms) within decoration spaces defined by product styles |
| NG-04 | Replacing the third-party service for first-time exports during a third-party outage | ADR-0005 explicitly notes this gap; legacy dual-run covers cached artifacts only |
| NG-05 | Real-time collaborative editing of decoration designs | Single-user editing model assumed; concurrency is handled via immutable revision snapshots |
<!-- DA:END block:north_star.non_goals -->

### Success Metrics

<!-- DA:BEGIN block:north_star.success_metrics -->
| Metric | Target | Measurement | Status |
|--------|--------|-------------|--------|
| Third-party call volume for new designs | 0 calls after cutover | API gateway metrics | Pending implementation |
| Legacy design orderability | 100% orderable within retention window | Order success rate monitoring | Pending ASM-0008 confirmation |
| Export determinism | Identical outputs for identical inputs | Golden-image comparison tests | Pending ASM-0005 resolution |
| Preview generation latency | Within SLO target (TBD) | P95 latency metrics | Pending OQ-0004 |
| Save operation latency | Within SLO target (TBD) | P95 latency metrics | Pending OQ-0004 |
| Constraint validation consistency | Zero drift between save-time and export-time validation | Shared constraint test vectors | Pending implementation |
| System availability | Per NFR-0002 targets | Uptime monitoring | Pending NFR-0002 targets |
<!-- DA:END block:north_star.success_metrics -->

