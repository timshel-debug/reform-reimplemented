# Architecture

> System architecture overview, components, deployment, and non-functionals.

---

## Overview

*Add your content here.*

---

## Generated Content (Managed)

> The sections below are managed by Discovery Agent.
> Do not edit content between the `DA:BEGIN` and `DA:END` markers.

### Overview

<!-- DA:BEGIN block:arch.overview -->
The Decoration Platform follows Clean Architecture (Ports and Adapters / Hexagonal Architecture, PAT-0001) with strict separation between domain logic and infrastructure concerns.

**Bounded Contexts:**

- **Decoration (Core)** -- design creation, revision management, constraint validation
- **Rendering/Export (Compute)** -- preview overlay generation, production export generation
- **Assets (Library)** -- approved artwork management and availability rules

**Architectural Style:**

- Domain logic is infrastructure-agnostic; all external dependencies are accessed through ports
- Adapters implement ports for specific infrastructure (RDS, S3, SQS, third-party API)
- The ConstraintEngine is a pure domain service with no infrastructure dependencies (PAT-0003)
- All design mutations produce immutable revision snapshots (PAT-0002)
<!-- DA:END block:arch.overview -->

### Components

<!-- DA:BEGIN block:arch.components -->
| Component | Type | Responsibility | Key Patterns |
|-----------|------|---------------|---------------|
| Decoration API | Stateless service | Validates requests, persists revisions, orchestrates jobs | Idempotency-Key (ADR-0003), RFC 7807 errors |
| Render Worker | Scale-out worker | Generates preview overlay images from revision data | Picks jobs from SQS, stores artifacts to S3 |
| Export Worker | Scale-out worker | Generates production exports with constraint re-validation | Deterministic output (ADR-0004), idempotent |
| ConstraintEngine | Pure domain service | Single source of constraint validation for save and export | PAT-0003, REQ-0003 |
| Style Read Adapter | Infrastructure adapter | Syncs style versions and decoration spaces from platform | ASM-0003, ADR-0006 |
| Legacy Adapter | Infrastructure adapter | Read-through cache for third-party legacy exports | ADR-0005, PAT-0005 |
| Asset Repository | Infrastructure adapter | Manages approved artwork storage and retrieval | S3-backed blob storage |
| Revision Repository | Infrastructure adapter | Persists and queries decoration revisions | RDS-backed, immutable records |
<!-- DA:END block:arch.components -->

### Deployment

<!-- DA:BEGIN block:arch.deployment -->
**Pattern:** Stateless API + scale-out Workers (ADR-0001)

**Components:**

- **Decoration API** -- stateless HTTP service; horizontally scalable behind load balancer; no local state
- **Render Workers** -- pool of workers consuming preview jobs from SQS; auto-scales based on queue depth
- **Export Workers** -- pool of workers consuming export jobs from SQS; auto-scales based on queue depth
- **RDS (PostgreSQL)** -- managed relational database for metadata (revisions, artifacts, jobs)
- **S3** -- object storage for binary artifacts (previews, exports, cached legacy exports)
- **SQS** -- managed message queue for async job distribution

**Scaling Strategy:**

- API scales horizontally based on request volume
- Workers scale independently based on queue depth (NFR-0001)
- Preview and export workers may share a pool or run as separate pools (deployment decision)
- RDS scales vertically; read replicas if needed for query load

**Environment Separation:**

- Standard environment promotion: dev -> staging -> production
- Feature flags for legacy dual-run behavior (ADR-0005)
<!-- DA:END block:arch.deployment -->

### Non Functionals

<!-- DA:BEGIN block:arch.non_functionals -->
| ID | Category | Requirement | Target | Status |
|----|----------|-------------|--------|--------|
| NFR-0001 | Scale | Preview/export workloads support horizontal scaling | Auto-scale workers by queue depth | Defined |
| NFR-0002 | Availability | System meets availability SLO | TBD (pending OQ-0004) | Pending |
| NFR-0004 | Security | Token-based authentication with role claims | Bearer token validation on all endpoints | Pending ASM-0002 |
| NFR-0005 | Retention | Artifacts stored for configurable retention window | >= 12 months default (pending ASM-0008) | Pending |
| -- | Latency | Save operation completes within SLO | TBD (pending OQ-0004) | Pending |
| -- | Latency | Preview generation completes within SLO | TBD (pending OQ-0004) | Pending |
| -- | Idempotency | All mutation endpoints support Idempotency-Key | Duplicate detection via stored keys (ADR-0003) | Defined |
| -- | Determinism | Export outputs are reproducible for identical inputs | Deterministic key = revision + style_version + export_profile (ADR-0004) | Defined (pending ASM-0005 for exact semantics) |
| -- | Observability | Structured logging, metrics, and tracing | Standard platform telemetry | Pending |
<!-- DA:END block:arch.non_functionals -->

