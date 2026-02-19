# Domain Pack

> Domain entities, workflows, events, and data sources.

---

## Overview

*Add your content here.*

---

## Generated Content (Managed)

> The sections below are managed by Discovery Agent.
> Do not edit content between the `DA:BEGIN` and `DA:END` markers.

### Entities

<!-- DA:BEGIN block:domain.entities -->
#### DecorationDesign (Aggregate Root)

Platform-level identifier for a decoration. Owns a sequence of immutable revisions.

- `design_id` (string) -- stable identifier provided by the platform (ASM-0009)
- `current_revision_id` (FK) -- pointer to the latest DecorationRevision
- `created_at`, `updated_at` (timestamp)

#### DecorationRevision (Aggregate)

Immutable snapshot of a decoration state. Each edit creates a new revision (REQ-0004, PAT-0002).

- `revision_id` (PK)
- `design_id` (FK)
- `style_version_id` -- pinned at save time (ADR-0006)
- `elements` -- collection of DecorationElement, organized per decoration space
- `constraint_snapshot` -- result of ConstraintEngine validation at save time
- `created_at` (timestamp)

#### DecorationElement (Value Object)

A single decoration item within a space on a revision.

- `element_id` (PK within revision)
- `type` -- `text` or `graphic`
- `space_id` -- the decoration space this element belongs to
- `content` -- text string (for text type) or asset reference (for graphic type)
- `transforms` -- position (x, y), scale, rotation

#### Asset (Aggregate)

Approved artwork available for use in graphic decoration elements.

- `asset_id` (PK)
- `name`, `description`
- `availability_rules` -- conditions under which the asset can be used
- `storage_ref` -- S3 object key for the artwork file

#### Artifact (Aggregate)

Generated output from preview or export jobs, stored in S3 with traceability metadata.

- `artifact_id` (PK)
- `type` -- `preview_overlay` or `production_export`
- `design_revision_id` (FK)
- `style_version_id`
- `export_profile_id` (for exports)
- `storage_ref` -- S3 object key
- `deterministic_key` -- hash of (design_revision_id + style_version_id + export_profile_id) per ADR-0004
- `created_at` (timestamp)
<!-- DA:END block:domain.entities -->

### Workflows

<!-- DA:BEGIN block:domain.workflows -->
#### WF-01: Save Flow

1. UI submits decoration state to Decoration API
2. API validates constraints via ConstraintEngine (REQ-0003)
3. API pins current `style_version_id` (ADR-0006)
4. API persists new immutable DecorationRevision (REQ-0004)
5. API returns revision ID to caller
6. Idempotency-Key prevents duplicate revisions (ADR-0003)

#### WF-02: Preview Flow

1. API receives preview request for a revision
2. API enqueues preview job to SQS with idempotent submission (ADR-0003)
3. Render Worker picks up job, generates overlay image
4. Worker stores artifact in S3 with traceability metadata
5. API returns artifact reference (sync or async per OQ-0007)

#### WF-03: Export Flow

1. API receives export request with revision + export profile
2. API checks artifact cache by deterministic key (ADR-0004)
3. On cache miss: enqueues export job to SQS
4. Export Worker validates constraints (same ConstraintEngine), generates production output
5. Worker stores artifact in S3 keyed by deterministic identity
6. API returns artifact reference

#### WF-04: Legacy Flow

1. API receives request for a legacy design export
2. API checks local artifact cache (read-through, ADR-0005)
3. On cache hit: returns cached artifact
4. On cache miss: calls third-party legacy service, stores result in cache
5. Note: first-time export during third-party outage is NOT covered (ADR-0005 known gap)
<!-- DA:END block:domain.workflows -->

### Events

<!-- DA:BEGIN block:domain.events -->
| Event | Trigger | Payload | Consumers |
|-------|---------|---------|----------|
| RevisionCreated | Save flow completes successfully | design_id, revision_id, style_version_id | Preview subsystem (auto-generate preview) |
| PreviewJobEnqueued | Preview requested or auto-triggered | revision_id, job_id, idempotency_key | Render Worker |
| PreviewArtifactStored | Render Worker completes preview | artifact_id, revision_id, storage_ref | API (cache update) |
| ExportJobEnqueued | Export requested and cache miss | revision_id, export_profile_id, job_id, idempotency_key | Export Worker |
| ExportArtifactStored | Export Worker completes export | artifact_id, deterministic_key, storage_ref | API (cache update) |
| LegacyCacheMiss | Legacy export requested, not in cache | design_id, legacy_ref | Legacy Adapter (calls third-party) |
| LegacyArtifactCached | Legacy Adapter stores retrieved export | artifact_id, design_id, storage_ref | API (cache update) |
| ConstraintViolationDetected | Validation fails at save or export time | revision_id, violations[] | API (error response) |
<!-- DA:END block:domain.events -->

### Data Sources

<!-- DA:BEGIN block:domain.data_sources -->
| Source | Type | Ownership | Purpose |
|--------|------|-----------|--------|
| RDS (PostgreSQL) | Relational database | Owned | Stores DecorationDesign, DecorationRevision, DecorationElement metadata, job records, and artifact metadata (ADR-0002) |
| S3 | Object storage | Owned | Stores binary artifacts -- preview overlays, production exports, and cached legacy exports (ADR-0002) |
| SQS | Message queue | Owned | Async job distribution for preview and export workers (ADR-0003) |
| Platform Style Read Model | Read-only API/sync | External (platform-owned) | Provides style definitions, decoration spaces, and version history for style version pinning (ASM-0003, ADR-0006) |
| Third-Party Legacy Service | External API | External (vendor) | Export generation for legacy designs during retention window; accessed only via Legacy Adapter read-through cache (ADR-0005) |
| Salesforce | External system | External | Source-of-truth for product configuration; out of scope for ownership change |
<!-- DA:END block:domain.data_sources -->

