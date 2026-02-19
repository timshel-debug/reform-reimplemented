# Test Strategy and Gates

## Coverage targets
TODO (not specified in inputs).

## Key tests

* Save/reload round-trip snapshot tests (REQ-0004). 
* Constraint vectors used in both save and export validation (REQ-0003). 
* Preview/export idempotency tests (REQ-0006/REQ-0007). 
* Legacy export integration tests + cache behavior (REQ-0008). 
* Rollout routing precedence tests (`env → tenant → style → user`) and rollback continuity tests (REQ-A-0001). 
* Legacy outage orderability test for uncached design plus coverage KPI validation (REQ-A-0002). 
* Legacy edit policy conformance tests (route determinism + metadata persistence + clone fidelity if enabled) (REQ-A-0003). 
* Prototype thin-slice acceptance test (`place → persist → preview → export`) with signoff checklist verification (REQ-A-0004). 

## Addendum test gates

* Gate-A1: Rollout flags audited and reversible with no stranded first-party designs.
* Gate-A2: Simulated third-party outage legacy order attempt succeeds when policy coverage is satisfied (cached/pre-generated artifact available).
* Gate-A3: Clone-to-new produces an owned design that is both editable and exportable, with required fidelity checks.
* Gate-A4: Prototype exit criteria met and signoff recorded.
* Gate-A5: Weekly KPI dashboard pack exists for rollout and legacy coverage governance (NFR-A-0001).

## Determinism regression
TODO pending ASM-0005 and OQ-0005. 
