# ADR-0007 — Legacy orderability policy (proposed)

## Status
Proposed (pending stakeholder signoff)

## Context
Legacy continuity currently relies on read-through cache and can fail for first-time legacy export when the third-party is unavailable.
This gap conflicts with addendum requirement for orderability under outage scenarios.


## Decision (Proposed Default)
Adopt scheduled pre-generation of legacy exports for in-scope cohorts, plus existing read-through on-demand behavior.

Policy shape:
* Scheduled backfill/pre-generation job for active legacy cohorts.
* Read-through export generation remains enabled when third-party is healthy.
* Coverage tracking and alerting for `% legacy designs with cached exports` and `max age of uncached legacy exports`.

## Consequences
* Closes the known read-through-only gap for first-time export during third-party outage.
* Adds operational burden: scheduling, coverage metrics, and retention management.
* Keeps migration available as fallback alternative, but not required for MVP.

## Validation / Exit Criteria
* Simulated third-party outage order test passes when coverage policy is satisfied.
* Coverage KPI and uncached-age KPI are visible in dashboards with alerts.
* Retention policy for cached legacy exports is explicit and enforced.

## Alternatives considered
* Full migration of legacy designs into owned storage.
* Alternate business fallback workflow with retained artifacts.
