# Operations

## Scalability

* API and workers scale horizontally (NFR-0001). 
* Queue depth and job age drive worker autoscaling (policy TODO).

## Availability

* New designs must not depend on third-party (NFR-0002). 
* Legacy path uses cached artifacts when available (ADR-0005).
* Legacy orderability policy for third-party outage must cover first-time export scenarios via pre-generation, migration, and/or alternate retained artifact workflow. 

## Rollout, cutover, and rollback

* Routing flags are evaluated in deterministic order: `environment → tenant → style → user`.
* Rollout supports dual-run window with explicit start/stop criteria and audited changes.
* Rollback reverts cohort routing without stranding first-party-created designs (view/export continuity required).
* All controls are represented with AWS-aligned primitives (e.g., AppConfig/parameter store + CloudTrail/audit store + CloudWatch alarms). 

## Retention

* Configurable retention window (ASM-0008 TODO). 
* Keep decoration revisions and export artifacts (or regeneratable inputs) for retention window (NFR-0005). 
* Legacy export artifact retention duration must be explicit and policy-enforced; minimum guidance is >= 12 months pending OQ-A-0004 confirmation. 

## Observability

* Structured logs, metrics, and tracing.
* SLO values are TODO (OQ-0004). 
* Required rollout/legacy policy KPIs:
	* Adoption split (`legacy` vs `owned`)
	* Export success/failure rates (`legacy`, `owned`)
	* Queue backlog and job latency (preview/export)
	* Legacy coverage percentage (`cached or migrated`)
* Dashboards define alert thresholds and a weekly reportable KPI pack for go/no-go governance. 

## Proposed KPI/SLO defaults (pending signoff)

* `% legacy designs with cached exports` (coverage KPI): proposed target >= 95% for active legacy cohort.
* `max age of uncached legacy exports`: proposed threshold <= 7 days for active legacy cohort.
* `legacy export success under outage drill`: proposed target 100% when coverage policy is satisfied.
* `rollout routing mismatch rate` (`provider metadata` vs `route outcome`): proposed target 0 critical mismatches.

## Outage playbook behavior (legacy third-party down)

* Step 1: Route orderability checks to cached/pre-generated artifact store.
* Step 2: If artifact missing, invoke bounded fallback policy response and enqueue prioritized backfill.
* Step 3: Surface user-facing bounded failure mode only when policy cannot be satisfied.
* Step 4: Trigger incident alert if uncached-age KPI breaches threshold.
