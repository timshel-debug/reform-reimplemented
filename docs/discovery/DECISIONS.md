# Decision Log

> Record of key decisions and their rationale.

---

## Overview

*Add your content here.*

---

## Generated Content (Managed)

> The sections below are managed by Discovery Agent.
> Do not edit content between the `DA:BEGIN` and `DA:END` markers.

### Log

<!-- DA:BEGIN block:decisions.log -->
| ID | Decision | Status | Rationale | Source |
|----|----------|--------|-----------|--------|
| ADR-0001 | Deploy as stateless Decoration API + scale-out Workers | Accepted | Preview and export workloads must support horizontal scaling without blocking interactive editing (NFR-0001) | docs/adr/ADR-0001-deploy-api-and-workers.md |
| ADR-0002 | Relational metadata (RDS) + object storage (S3) for blobs | Accepted | Must persist editable state and binary artifacts for months; relational DB for queryable metadata, S3 for cost-effective blob storage | docs/adr/ADR-0002-relational-plus-object-storage.md |
| ADR-0003 | SQS queue-based async jobs with idempotent submission | Accepted | Decouples compute-heavy rendering from API latency; Idempotency-Key header prevents duplicate job creation | docs/adr/ADR-0003 |
| ADR-0004 | Export determinism envelope: identity = revision + style_version + export_profile | Accepted | Exports must be reproducible for a saved design state; deterministic key enables artifact caching | docs/adr/ADR-0004-export-determinism-envelope.md |
| ADR-0005 | Legacy dual-run with read-through artifact cache | Accepted | Legacy designs must remain orderable for retention window; cache reduces third-party call volume; known gap for first-time export during outage | docs/adr/ADR-0005-legacy-dualrun-and-cache.md |
| ADR-0006 | Style version pinning with explicit compatibility mapping | Accepted | Styles/spaces are versioned; saved designs must remain compatible when configurations change | docs/adr/ADR-0006-style-version-pinning-and-mapping.md |
| ADR-0008 | Legacy edit policy: deterministic behavior in UI and API | Accepted | Legacy edit behavior must be deterministic and reflected in UI routing, API contracts, and persisted metadata | docs/adr/ADR-0008-legacy-edit-policy.md |
<!-- DA:END block:decisions.log -->

