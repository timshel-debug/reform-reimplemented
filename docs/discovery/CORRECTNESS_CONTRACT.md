# Correctness Contract

> Invariants, gates, and regression tests.

---

## Overview

*Add your content here.*

---

## Generated Content (Managed)

> The sections below are managed by Discovery Agent.
> Do not edit content between the `DA:BEGIN` and `DA:END` markers.

### Invariants

<!-- DA:BEGIN block:correctness.invariants -->
| ID | Invariant | Enforcement | Source |
|----|-----------|-------------|--------|
| INV-01 | DecorationRevisions are immutable once persisted | No UPDATE/DELETE on revision rows; edits create new revisions | REQ-0004, PAT-0002 |
| INV-02 | ConstraintEngine produces identical results at save-time and export-time for the same inputs | Single ConstraintEngine implementation; shared test vectors | REQ-0003, PAT-0003 |
| INV-03 | Export identity is deterministic: same (revision + style_version + export_profile) always maps to same artifact | Deterministic key computation; artifact cache keyed by this tuple | ADR-0004 |
| INV-04 | Every revision pins a valid style_version_id at save time | Save flow validates style_version_id existence before persisting | ADR-0006 |
| INV-05 | Idempotent job submission: same Idempotency-Key never creates duplicate jobs or revisions | Check-before-enqueue pattern with stored idempotency keys | ADR-0003 |
| INV-06 | Legacy artifacts are cached on first retrieval; subsequent requests never call third-party for the same artifact | Read-through cache with cache-aside invalidation only on explicit purge | ADR-0005 |
| INV-07 | No new designs depend on third-party service | API routing ensures new design flows never invoke Legacy Adapter | NFR-0002 |
<!-- DA:END block:correctness.invariants -->

### Gates

<!-- DA:BEGIN block:correctness.gates -->
| Gate | Command | Pass Criteria | Phase |
|------|---------|---------------|-------|
| Build | `dotnet build --nologo -p:TreatWarningsAsErrors=true` | Zero warnings, exit code 0 | Pre-commit |
| Test | `dotnet test --nologo` | All tests pass, exit code 0 | Pre-commit |
| Constraint Consistency | Run shared constraint test vectors against ConstraintEngine | Save-time and export-time results match for all vectors | CI |
| Idempotency | Submit same Idempotency-Key twice; verify single revision/job created | No duplicates in database | CI |
| Export Determinism | Generate export twice for same inputs; compare artifacts | Byte-identical output (pending ASM-0005 for exact semantics) | CI |
| Legacy Cache | Simulate third-party call followed by cache hit; verify no second external call | Single external call per unique legacy artifact | CI |
<!-- DA:END block:correctness.gates -->

### Regression Tests

<!-- DA:BEGIN block:correctness.regression_tests -->
| ID | Test | Validates | Priority |
|----|------|-----------|----------|
| RT-01 | Save/reload snapshot test: save a revision, reload it, verify all fields match | REQ-0004 (immutable revisions) | High |
| RT-02 | Constraint validation vectors: run same inputs through ConstraintEngine at save and export paths | REQ-0003 (no drift) | High |
| RT-03 | Idempotent save: submit same Idempotency-Key twice, verify single revision created | ADR-0003 | High |
| RT-04 | Idempotent job submission: enqueue same job key twice, verify single job exists | ADR-0003 | High |
| RT-05 | Export determinism: generate export for same inputs twice, compare artifacts | ADR-0004 | High (pending ASM-0005) |
| RT-06 | Style version pinning: save revision, update style, verify revision still uses pinned version | ADR-0006 | Medium |
| RT-07 | Legacy cache hit: retrieve legacy export, verify cached on second request without third-party call | ADR-0005 | Medium |
| RT-08 | Legacy cache miss during outage: simulate third-party failure, verify 503 returned | ADR-0005 (known gap) | Medium |
| RT-09 | Constraint violation at save: submit invalid elements, verify 422 with CONSTRAINT_VIOLATION error code | REQ-0003 | High |
| RT-10 | New design routing: verify new design flow never invokes Legacy Adapter | NFR-0002 | High |
<!-- DA:END block:correctness.regression_tests -->

