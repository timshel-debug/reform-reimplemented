# SPEC-DECOR-0001 Addendum A — Rollout, Legacy Continuity, Prototyping, and Agentic Roadmap

## 0) Purpose

This addendum introduces minimum additional requirements required for the solution design to fully conform to the original planning workshop objectives, without materially expanding scope beyond what was explicitly requested (transition/legacy handling, rollout/prototyping, and future-facing agentic assistance).

## 1) Addendum Functional Requirements (REQ-A)

### REQ-A-0001 — Rollout, Cutover, and Rollback Controls

The solution MUST define and implement a controlled rollout mechanism that supports:

* Enabling the first-party decoration editor per environment, shop/tenant, product style, and/or user cohort.
* Routing decisions for new designs vs legacy designs (third-party vs first-party paths) based on a deterministic policy.
* A rollback strategy that can revert to legacy behaviour for impacted cohorts without data loss for designs created during the rollout window.

#### Acceptance Criteria

* AC-REQ-A-0001-01: Feature flags exist for editor routing (legacy vs first-party) with documented evaluation order (env → tenant → style → user).
* AC-REQ-A-0001-02: A dual-run window policy is documented with explicit start/stop criteria.
* AC-REQ-A-0001-03: Rollback does not strand users: designs created under first-party remain viewable/exportable even if editor routing is reverted.
* AC-REQ-A-0001-04: Rollout/rollback actions are auditable (who/when/what flag change).

### REQ-A-0002 — Legacy Design Orderability Under Third-Party Outage

The solution MUST define and implement a policy ensuring that legacy designs remain orderable even if the third-party decoration service is unavailable at order time.

This MUST be satisfied by one (or a combination) of:

* Pre-generation and storage of required production exports for legacy designs (scheduled or triggered), or
* Migration of legacy decoration data into first-party storage, or
* A formally documented business policy that guarantees orderability via alternate retained artifacts or workflows (must be automated where feasible).

#### Acceptance Criteria

* AC-REQ-A-0002-01: For a legacy design that has not been exported previously, ordering can still complete when the third-party is down (testable scenario).
* AC-REQ-A-0002-02: The system can demonstrate a measurable coverage strategy (e.g., all legacy designs touched in last N months have cached exports or equivalent).
* AC-REQ-A-0002-03: The retention duration for legacy exports/artifacts is explicitly specified (ties to retention policy).
* AC-REQ-A-0002-04: A user-facing failure mode is defined if the policy cannot be met (should be rare and bounded).

### REQ-A-0003 — Explicit Legacy Editing Behaviour

The solution MUST explicitly specify whether legacy designs are:

* Editable in the new first-party editor, OR
* Editable only via the legacy tool path, OR
* Non-editable (order-only), with a defined clone to new workflow.

#### Acceptance Criteria

* AC-REQ-A-0003-01: The UI routing rules for legacy designs are documented and deterministic.
* AC-REQ-A-0003-02: If clone to new is supported, the resulting new design is first-party owned and preserves necessary visual fidelity.
* AC-REQ-A-0003-03: The chosen policy is reflected in API contracts and persisted metadata (e.g., decoration_provider = legacy|owned).

### REQ-A-0004 — Prototyping Plan for Decoration Tool Complexity

The solution MUST include a prototyping plan to validate the complexity and feasibility of first-party decoration tooling and export generation, including:

* A minimal interactive editor prototype (text + graphic placement) within decoration spaces
* A prototype of production export generation for at least one space and one process type (e.g., embroidery)
* A clear evaluation checklist and exit criteria to proceed to build

#### Acceptance Criteria

* AC-REQ-A-0004-01: Prototype scope, timeline, and success criteria are documented.
* AC-REQ-A-0004-02: Prototype validates at least one complete path: place elements → persist → render preview → generate export artifact.
* AC-REQ-A-0004-03: Identified unknowns/risks are captured with mitigation actions.

### REQ-A-0005 — Agentic Assistance (Non-Blocking Roadmap Commitment)

The solution MUST include an explicit appendix that enumerates potential agentic/AI-assisted enhancements to streamline design, without blocking MVP delivery. Minimum includes:

* Colour/palette guidance based on early selections
* Reusable templates (e.g., school year or repeat ordering patterns)
* Assisted composition for decoration layouts within constraints

#### Acceptance Criteria

* AC-REQ-A-0005-01: Clearly marked non-MVP with a staged roadmap (Phase 2+).
* AC-REQ-A-0005-02: Each idea includes: entry point in UX, required inputs, and constraints/safety considerations.
* AC-REQ-A-0005-03: Architecture impact is documented (where signals/logs/models plug in, without committing to implementation now).

## 2) Addendum Non-Functional Requirements (NFR-A)

### NFR-A-0001 — Operational Observability for Rollout and Legacy Policy

The solution MUST provide monitoring/metrics sufficient to validate rollout safety and legacy orderability:

* Adoption split (legacy vs owned)
* Export success/failure rates (legacy and owned)
* Queue backlogs / job latencies for preview/export
* % of legacy designs with cached exports (or migrated)

#### Acceptance Criteria

* AC-NFR-A-0001-01: Dashboards/metrics are defined with alert thresholds.
* AC-NFR-A-0001-02: A weekly reportable KPI set is specified (sufficient for go/no-go decisions).

## 3) Addendum Constraints (CON-A)

### CON-A-0001 — AWS Alignment

All rollout, storage, compute, queueing, and observability approaches MUST be expressed using AWS primitives (or cloud-agnostic abstractions deployable on AWS).

## 4) Addendum Open Questions (OQ-A) — Must be answered to exit design

* OQ-A-0001: Which legacy orderability approach is chosen (pre-generate exports vs migrate vs alternate policy)?
* OQ-A-0002: What is the explicit legacy edit policy (edit/legacy-only/clone-to-new)?
* OQ-A-0003: What are the prototype exit criteria and who signs them off?
* OQ-A-0004: What retention window is mandated (minimum >= 12 months implied; confirm exact)?
