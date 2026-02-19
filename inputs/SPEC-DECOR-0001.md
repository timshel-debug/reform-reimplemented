# SPEC-DECOR-0001 — Owned Decoration Platform (Requirement Specification)

## 0) Document Control

* **Spec ID**: SPEC-DECOR-0001
* **System**: Design Lab / Decoration Platform
* **Status**: Draft (requirements isolated; open questions tracked)
* **Primary source**: Provided transcript + attached PDF (planning workshop)

---

## 1) Purpose

Own the end-to-end **decoration management** capability (data + APIs + editor tooling + rendering/export) to reduce dependency risk on the current third-party decoration service, while preserving continuity for **legacy saved designs** that may be ordered long after creation.

---

## 2) Scope

### 2.1 In scope

* First-party **decoration editor** capability (minimum: text + graphics) operating within defined decoration spaces.
* First-party **decoration data persistence** (store elements, transforms, styling, assets).
* **Preview rendering** for interactive editing (per-side overlays).
* **Production export generation** suitable for factory/ordering workflows.
* **Legacy continuity** strategy implementation (at least order/export continuity; editing behaviour is TBD).

### 2.2 Out of scope (explicit)

* Garment **colouring pipeline** ownership changes (SVG colouring, fabric→palette) beyond integration.
* Replacing **Salesforce** as product configuration source-of-truth.
* Any changes to the third-party service (assumed immutable during transition).

---

## 3) Definitions

* **Product Style**: Versioned product configuration defining colour regions and decoration spaces.
* **Colour Region**: SVG-grouped paths (by class/group) that can be coloured; palettes may vary by fabric.
* **Decoration Space**: Bounded region (e.g., chest-left) with geometry and constraints (process/application type).
* **Decoration Element**: Item placed into a decoration space (text or graphic) with transforms/styling.
* **Legacy Design**: A saved design whose decoration data lives in the third-party service.
* **Production Export**: Print/production-ready artifact (e.g., PDF) derived from a design for ordering/factory use.

---

## 4) Actors

* **Customer**: creates/edits designs; saves; returns later; orders.
* **Admin/Operations**: maintains products/spaces/process constraints and asset libraries (where applicable).
* **Platform**: stores design records; orchestrates preview/export; integrates Salesforce sync and ordering flows.

---

## 5) Functional Requirements (REQ)

### REQ-0001 — Consume Product Style + Decoration Space Definitions

The solution MUST consume versioned product style configuration, including decoration spaces and their constraints.

* **Acceptance Criteria**

  * AC-REQ-0001-01: For a given style version, list all decoration spaces per side with geometry + constraints.
  * AC-REQ-0001-02: A saved design can be reloaded using the same space definitions or an explicit version mapping.

---

### REQ-0002 — Provide First-Party Decoration Editor (Minimum)

The solution MUST provide first-party editing for decoration spaces supporting at minimum:

* Text elements (content + basic styling)
* Graphic elements (asset placement)
* Positioning + scaling within the decoration space
* **Acceptance Criteria**

  * AC-REQ-0002-01: Add/edit/remove text in a decoration space.
  * AC-REQ-0002-02: Add/edit/remove a graphic asset in a decoration space.
  * AC-REQ-0002-03: Move/scale elements and preview the result in-context.

---

### REQ-0003 — Enforce Decoration Space Constraints

The solution MUST enforce per-space constraints (process/application type and implied restrictions such as palette/tool limits).

* **Acceptance Criteria**

  * AC-REQ-0003-01: Editor restricts actions/options based on the space’s process/application type.
  * AC-REQ-0003-02: Export pipeline applies the same constraints as the editor (no “invalid export” drift).

---

### REQ-0004 — Persist First-Party Decoration Data

The solution MUST store sufficient decoration data to reload, re-edit, preview, and export without reliance on the third-party service for **new designs**.
Minimum persisted data includes:

* Stable IDs: design, space, element
* Element type (text/graphic)
* Transforms (position/scale; rotation if supported)
* Text content + styling OR asset reference + styling
* Process metadata required for export
* **Acceptance Criteria**

  * AC-REQ-0004-01: Saving persists a complete editable representation.
  * AC-REQ-0004-02: Reloading restores identical editor state.

---

### REQ-0005 — Integrate with Full Design Save/Load

The platform MUST be able to save/load a complete design composed of:

* Garment colour configuration (existing capability)
* Decoration configuration (new first-party capability)
* **Acceptance Criteria**

  * AC-REQ-0005-01: Saved design links to first-party decoration data needed for reload/edit/export.
  * AC-REQ-0005-02: Resume-edit works after long gaps without data loss.

---

### REQ-0006 — Generate Preview Rendering

The solution MUST generate per-side previews compositing decorations onto garment views for interactive UX and review.

* **Acceptance Criteria**

  * AC-REQ-0006-01: Preview/overlay is generated for each relevant side from the saved design state.
  * AC-REQ-0006-02: Supports repeated edit/save cycles reliably.

---

### REQ-0007 — Generate Production Exports

The solution MUST generate production-ready exports for ordering/manufacturing workflows.

* **Acceptance Criteria**

  * AC-REQ-0007-01: For any orderable design, generate a production export artifact (format TBD).
  * AC-REQ-0007-02: Export generation uses the saved design state as the source of truth.

---

### REQ-0008 — Legacy Design Continuity

The solution MUST preserve continuity for legacy designs created using the third-party service, such that customers can still order (and obtain required production exports) long after creation.

* **Acceptance Criteria**

  * AC-REQ-0008-01: Legacy designs remain orderable for the defined retention window (see ASM-0008).
  * AC-REQ-0008-02: Required production exports for legacy designs can be obtained during ordering.
* **Note**: Whether legacy designs are editable in the new editor is explicitly tracked as OQ-0002.

---

### REQ-0009 — Artwork Asset Library (Minimum)

The solution MUST support selecting approved graphic assets for placement as decoration elements (management model TBD).

* **Acceptance Criteria**

  * AC-REQ-0009-01: User can browse/select an approved asset and place it as a graphic element.
  * AC-REQ-0009-02: Admin/Operations can manage asset availability (mechanism TBD).

---

### REQ-0010 — Versioning & Traceability

The solution MUST preserve traceability between product style versions, saved designs, and generated exports.

* **Acceptance Criteria**

  * AC-REQ-0010-01: Saved designs are explicitly associated with a product style/version identifier.
  * AC-REQ-0010-02: Exports include sufficient metadata to trace back to the originating design/style version.

---

## 6) Non-Functional Requirements (NFR)

### NFR-0001 — Scale

Must support hundreds–thousands of saves per day and frequent edit cycles.

* AC-NFR-0001-01: Architecture supports horizontal scaling for preview/export workloads.
* AC-NFR-0001-02: Persistence layer supports expected write/read volume without manual intervention.

### NFR-0002 — Availability / De-risk Third-Party Dependency

New designs must not be blocked by third-party outages.

* AC-NFR-0002-01: New designs can be saved/reloaded/exported without third-party dependency.
* AC-NFR-0002-02: Legacy outage mitigation policy exists (tracked in OQ-0001/OQ-0008).

### NFR-0003 — Interactive Performance

Usable interactive editing experience under expected concurrency (targets TBD).

* AC-NFR-0003-01: Preview and save latencies meet defined SLOs (see OQ-0004).

### NFR-0004 — Security & Access Control

Access to designs and admin controls must be restricted appropriately.

* AC-NFR-0004-01: Only authorized users can access/modify their designs.
* AC-NFR-0004-02: Admin-only controls for assets/constraints/settings (where applicable).

### NFR-0005 — Data Retention

Designs must be retained long enough to support delayed group ordering.

* AC-NFR-0005-01: Retention meets or exceeds the business-required window (see ASM-0008).
* AC-NFR-0005-02: Export artifacts or regeneratable inputs persist for the retention window.

---

## 7) Constraints (CON)

### CON-0001 — Salesforce as Source-of-Truth

Product styles and decoration spaces are configured via Salesforce and synced to the platform (solution must integrate, not replace).

### CON-0002 — Third-Party System Is Immutable

The third-party service cannot be modified; transition must work with it as-is.

### CON-0003 — Version Coupling

Product style changes and decoration-space definitions are versioned; saved designs must remain compatible with the relevant version.

---

## 8) Assumptions (ASM) — Statused Truth Set

### ASM-0001 — Deployment Target

**AWS** is the cloud target for deployment and operations.

* **Status**: **Confirmed**

### ASM-0002 — Identity Provider & Claims

Platform provides authenticated identity and role claims (Customer vs Admin/Operations).

* **Status**: **TODO** (unanswered)

### ASM-0003 — Salesforce Sync Access Path Exists

Platform already exposes a way to read style versions and decoration spaces (via API/read model).

* **Status**: **Answered** (Salesforce → platform sync described)

### ASM-0004 — Colouring Pipeline Exists Upstream

Garment colouring is already supported; decoration preview composes over existing garment visuals.

* **Status**: **Answered**

### ASM-0005 — Export Determinism Definition

“Idempotent/deterministic export” semantics and version pinning are required.

* **Status**: **TODO** (not specified; only “export for factory use” is explicit)

### ASM-0006 — Fonts/Assets Licensing

Fonts and artwork assets are licensed for rendering and distribution.

* **Status**: **TODO** (unanswered)

### ASM-0007 — Constraints Are Rule-Expressible

Process/application constraints can be represented as a finite rule set (not requiring a complex rules engine).

* **Status**: **PARTIAL / TODO** (constraints exist; complexity not specified)

### ASM-0008 — Retention Window Policy

Retention covers at least 6–12 months; exact policy duration is configurable and must be confirmed.

* **Status**: **PARTIAL / TODO** (6–12 months implied; exact window not confirmed)

---

## 9) Open Questions (OQ)

### OQ-0001 — Legacy Strategy

Migrate legacy decoration data vs dual-run vs cutoff policy?

### OQ-0002 — Legacy Edit Support

Must legacy designs be editable in the new editor, or only orderable/exportable?

### OQ-0003 — MVP Feature Set Beyond Text/Graphics

Do we need layers, rotation, multi-colour embroidery rules, advanced typography, grouping, etc.?

### OQ-0004 — Performance Targets

What are the SLOs for preview latency, save latency, and concurrency?

### OQ-0005 — Export Format Requirements

Exact export artifact format/structure (PDF layout, per-space outputs, factory integration requirements)?

### OQ-0006 — Space Definition Snapshotting

Snapshot space definitions into each saved design revision vs reference with compatibility mapping?

### OQ-0007 — Preview API Mode

Synchronous preview endpoints vs async generation with caching acceptable?

### OQ-0008 — Export Timing Strategy

Exports generated only at order time vs also pre-generated during design save (or on explicit user action)?

---

## 10) Explicit Non-Requirements (NR)

These are explicitly not required for the initial solution design unless later promoted to requirements:

* AI/agent-assisted design suggestions or automation.
* UX redesign beyond parity with current decoration workflows.
* Prototyping approach/vendor selection (unless needed to validate OQs).
