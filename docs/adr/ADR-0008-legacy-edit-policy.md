# ADR-0008 — Legacy edit policy (proposed)

## Status
Proposed (pending stakeholder signoff)

## Context
Legacy edit behavior must be deterministic and reflected in UI routing, API contracts, and persisted metadata.


## Decision (Proposed Default)
Use clone-to-new for editing:
* Legacy designs remain order/export capable via legacy continuity policy.
* Editing a legacy design creates a new owned design clone.
* Subsequent edits occur only in first-party owned editor path.

## Consequences
* Avoids in-place mutation of third-party legacy models.
* Aligns with immutable revision and provider metadata patterns (`decoration_provider=legacy|owned`).
* Requires clone endpoint/workflow and visual fidelity validation criteria.

## Validation / Exit Criteria
* Clone operation produces owned design that is editable and exportable.
* Route decisions for legacy vs owned are deterministic and auditable.
* Clone fidelity checks pass against required visual baseline.

## Alternatives considered
* Legacy-only editing path.
* Direct conversion/import to editable owned model without explicit clone UX.
* Non-editable legacy order-only policy.
