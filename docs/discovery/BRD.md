# Business Requirements Document

> Executive summary, user journeys, and requirements overview.

---

## Overview

*Add your content here.*

---

## Generated Content (Managed)

> The sections below are managed by Discovery Agent.
> Do not edit content between the `DA:BEGIN` and `DA:END` markers.

### Executive Summary

<!-- DA:BEGIN block:brd.executive_summary -->
The Decoration Platform project replaces the current dependency on a third-party decoration service for garment customization workflows. Today, all decoration design creation, preview rendering, and production export generation flow through an external vendor, creating single-point-of-failure risk for the ordering pipeline.

The owned platform will handle the complete decoration lifecycle: design creation with constraint validation, preview overlay rendering, and deterministic production export generation. Legacy designs created on the third-party service will remain orderable during a configurable retention window (default 12 months, pending ASM-0008 confirmation) via a read-through artifact cache.

The architecture follows Clean Architecture principles with a stateless API layer and horizontally scalable worker pool for compute-intensive preview and export operations. Immutable design revisions with style version pinning ensure that saved designs remain stable even as product configurations evolve.

**Business value:**

- Eliminates vendor lock-in for a critical ordering workflow
- Enables independent scaling and feature velocity for decoration capabilities
- Maintains zero disruption to existing customer orders during transition
<!-- DA:END block:brd.executive_summary -->

### User Journeys

<!-- DA:BEGIN block:brd.user_journeys -->
#### UJ-01: Designer Creates New Decoration

1. Designer opens the decoration editor for a product/style
2. Platform loads current style version with decoration spaces
3. Designer adds text and graphic elements to available spaces
4. Editor validates constraints in real-time via ConstraintEngine
5. Designer saves -> API creates immutable DecorationRevision, pins style_version_id
6. Preview is auto-generated and displayed

#### UJ-02: Designer Edits Existing Decoration

1. Designer opens an existing decoration design
2. Platform loads the current revision with its pinned style version
3. Designer modifies elements within constraint boundaries
4. Designer saves -> new immutable revision is created (prior revision preserved)
5. Updated preview is generated

#### UJ-03: Order Placed with New Decoration

1. Customer completes order containing a decorated product
2. System requests production export for the design revision
3. Export Worker validates constraints and generates deterministic output
4. Export artifact is stored and linked to order

#### UJ-04: Order Placed with Legacy Decoration

1. Customer completes order containing a legacy decorated product
2. System requests export via Legacy Adapter
3. Adapter checks local artifact cache
4. On cache miss: calls third-party service, caches result
5. Export artifact is returned for order fulfillment

#### UJ-05: Legacy Design During Third-Party Outage

1. Customer places order with legacy decoration
2. Adapter checks cache -> hit: order proceeds normally
3. Adapter checks cache -> miss: third-party call fails -> order blocked (known gap per ADR-0005)
<!-- DA:END block:brd.user_journeys -->

### Requirements Overview

<!-- DA:BEGIN block:brd.requirements_overview -->
#### Functional Requirements

| ID | Requirement | Source | Status |
|----|-------------|--------|--------|
| REQ-0001 | New designs created and exported entirely on owned platform | SPEC-DECOR-0001 | Defined |
| REQ-0003 | Same ConstraintEngine validates at save-time and export-time | SPEC-DECOR-0001 | Defined |
| REQ-0004 | Design revisions are immutable snapshots | SPEC-DECOR-0001 | Defined |
| REQ-0008 | Legacy designs remain orderable for retention window | SPEC-DECOR-0001 | Defined |
| REQ-2000 | Retention window policy -- confirm storage semantics | ASM-0008 | TODO |
| REQ-2001 | Platform design identity -- confirm stable ID model | ASM-0009 | TODO |
| REQ-2002 | Third-party legacy export API reliability assessment | OQ-0009 | TODO |
| REQ-2003 | DecorationRevision must satisfy constraint engine at save time | Domain Model | Defined |
| REQ-2004 | Style/space versioning -- saved designs must remain compatible | ADR-0006 | Defined |

#### Non-Functional Requirements

| ID | Requirement | Source | Status |
|----|-------------|--------|--------|
| NFR-0001 | Horizontal scaling for preview/export workloads | SPEC-DECOR-0001 | Defined |
| NFR-0002 | System availability targets | SPEC-DECOR-0001 | Pending targets |
| NFR-0004 | Security -- token-based auth with role claims | SPEC-DECOR-0001 | Pending ASM-0002 |
| NFR-0005 | Retention -- artifact storage for configured window | SPEC-DECOR-0001 | Pending ASM-0008 |
<!-- DA:END block:brd.requirements_overview -->

