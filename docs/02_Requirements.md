# Requirements

This section restates requirements from the base specification and Addendum A, and maps them to solution components, tests, and ops controls. 

## Functional Requirements (REQ)

* REQ-0001..REQ-0010 as defined in `inputs/SPEC-DECOR-0001.md`. 
* REQ-A-0001..REQ-A-0005 as defined in `inputs/SPEC-DECOR-0001-ADDENDUM-A.md`. 

## Non-Functional Requirements (NFR)

* NFR-0001..NFR-0005 as defined in `inputs/SPEC-DECOR-0001.md`. 
* NFR-A-0001 as defined in `inputs/SPEC-DECOR-0001-ADDENDUM-A.md`. 

## Addendum constraints and design-exit questions

* CON-A-0001 (AWS alignment) constrains rollout/storage/compute/observability decisions. 
* OQ-A-0001..OQ-A-0004 are mandatory to close before design exit. 

## Traceability matrix

| Requirement | Primary components | Tests (min) | Ops controls (min) |
|---|---|---|---|
| REQ-0001 | Style Read Adapter; Style Space Cache | Contract tests for style-space DTO; mapping tests | Cache hit/miss metrics; sync lag alarms |
| REQ-0002 | Editor UI (owned); Decoration API | E2E editor tests; API validation tests | Frontend error telemetry; API 4xx/5xx dashboards |
| REQ-0003 | Constraint Engine (domain); Export Validator | Unit tests for constraints; export validation tests | Constraint violation counters; audit log |
| REQ-0004 | Decoration Store (RDB); Asset Store (object) | Persistence integration tests; round-trip snapshot tests | Backups; retention job logs |
| REQ-0005 | Platform integration contract; Decoration API | Contract tests (platform↔decoration); E2E resume-edit | Correlation IDs; audit trail completeness |
| REQ-0006 | Preview Renderer; Preview Cache | Golden-image tests (tolerance rules TBD); load tests | Render latency SLO metrics; queue depth |
| REQ-0007 | Export Generator; Artifact Store | Deterministic export snapshot tests; idempotency tests | Export job SLA metrics; artifact integrity checks |
| REQ-0008 | Legacy Adapter; Legacy Artifact Cache | Legacy export integration tests (mocked if needed) | Third-party dependency health; fallback policy alarms |
| REQ-0009 | Asset Library API; Admin UI | CRUD tests; access control tests | Admin audit logs; asset scan pipeline (TODO) |
| REQ-0010 | Versioning model; Metadata stamping | Version association tests; trace lookups | Immutable audit fields; export metadata checks |
| REQ-A-0001 | Routing Policy Engine; Feature Flag Store; Rollout Admin Controls | Flag evaluation order tests; rollback scenario tests; audit-log tests | Cohort routing dashboards; flag-change audit trail; rollback runbook |
| REQ-A-0002 | Legacy Continuity Policy Service; Legacy Export Cache Coverage Jobs | Third-party outage orderability test for uncached legacy design; cache coverage tests | Coverage KPI (`% legacy cached/migrated`); outage fallback alarms; retention monitors |
| REQ-A-0003 | Legacy Design Routing; Clone-to-New Workflow; Provider Metadata | Deterministic UI/API routing tests; clone fidelity tests; metadata persistence tests | Legacy-policy decision logs; mismatch alerting (`provider` vs route) |
| REQ-A-0004 | Prototype Workstream (Editor + Preview + Export thin slice) | End-to-end prototype path test (place → persist → preview → export); prototype exit checklist validation | Prototype risk register updates; go/no-go signoff records |
| REQ-A-0005 | Agentic Appendix and Extensibility Seams (non-MVP) | Roadmap documentation completeness checks; safety/constraint checklist review | Phase gating: no MVP dependency alerts; model-signal instrumentation plan |
| NFR-A-0001 | Observability Layer (metrics/logs/traces + weekly KPI pack) | Metric emission tests; dashboard query validation | Alert thresholds for adoption split, export success, queue latency, and legacy cache coverage |

## Addendum end-to-end mapping

| Requirement | Doc section | API endpoint/job | Test(s) | Ops metric/alert |
|---|---|---|---|---|
| REQ-A-0001 | `docs/03_Architecture.md` (routing policy flow), `docs/00_Assumptions_and_OpenQuestions.md` (OQ-A-0001/OQ-A-0002 governance) | `GET /routing/policy/evaluate`, `PATCH /admin/routing/flags/{scope}` | Routing precedence tests; rollback continuity tests; flag audit tests | Cohort adoption split dashboard; routing mismatch alert |
| REQ-A-0002 | `docs/08_Operations.md` (outage playbook + KPI defaults), `docs/adr/ADR-0007-legacy-orderability-policy.md` | `POST /admin/legacy/coverage/backfill`, `GET /legacy/coverage`, `legacy-coverage-backfill` schedule | Simulated outage order test when coverage satisfied; coverage KPI validation | `% legacy cached` KPI; uncached-age threshold alert; outage failure alert |
| REQ-A-0003 | `docs/04_Domain_Model.md` (`decoration_provider`), `docs/adr/ADR-0008-legacy-edit-policy.md` | `POST /designs/{designId}/clone`, routing evaluate endpoint with `edit_policy` | Clone-to-new fidelity test; owned editability test; deterministic route + metadata tests | Clone success rate; provider/route mismatch alert |
| REQ-A-0004 | `docs/03_Architecture.md` (prototyping workstream), `docs/10_Implementation_Plan.md` (SLICE-0011) | Prototype thin-slice orchestration job/workflow | Thin-slice acceptance (`place → persist → preview → export`); signoff checklist gate | Prototype risk burndown trend; go/no-go signoff audit |
| REQ-A-0005 | `docs/03_Architecture.md` (agentic appendix), `docs/10_Implementation_Plan.md` (SLICE-0012) | No MVP endpoint commitment; Phase 2+ extension seams only | Roadmap completeness review; safety constraints review | MVP dependency guardrail alert (agentic features not blocking) |
| NFR-A-0001 | `docs/08_Operations.md` (required KPI pack + thresholds) | Metrics pipeline + dashboard jobs | Metric emission and dashboard query validation | Weekly KPI pack completeness + threshold alerts |

Notes:
* Specific metrics/SLO values are TODO until OQ-0004 is resolved. 
* Export format and golden test criteria depend on OQ-0005. 
* Addendum metrics and go/no-go thresholds are required for rollout safety and legacy orderability governance. 
