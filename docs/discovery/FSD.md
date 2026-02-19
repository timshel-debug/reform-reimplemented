# Functional Specification Document

> System behaviour, API contracts, and data contracts.

---

## Overview

*Add your content here.*

---

## Generated Content (Managed)

> The sections below are managed by Discovery Agent.
> Do not edit content between the `DA:BEGIN` and `DA:END` markers.

### System Behaviour

<!-- DA:BEGIN block:fsd.system_behaviour -->
#### SB-01: Save Decoration Revision

- **Trigger:** Client submits decoration state with Idempotency-Key header
- **Preconditions:** Valid authentication token, design_id exists, style_version_id is current or mapped
- **Processing:**
  1. Check idempotency key -> if duplicate, return existing revision
  2. Validate all elements against ConstraintEngine for the target style/spaces
  3. Pin style_version_id to current version at save time
  4. Persist new immutable DecorationRevision with elements
  5. Emit RevisionCreated event
- **Postconditions:** New revision is queryable; prior revisions remain unchanged
- **Error cases:** Constraint violation (terminal, 422), style version not found (terminal, 404)

#### SB-02: Generate Preview

- **Trigger:** RevisionCreated event or explicit preview request
- **Processing:**
  1. Check artifact cache by revision_id
  2. On miss: enqueue preview job (idempotent submission per ADR-0003)
  3. Worker renders overlay image from revision elements
  4. Worker stores artifact in S3, updates metadata in RDS
- **Delivery:** Sync return or async poll (pending OQ-0007)

#### SB-03: Generate Production Export

- **Trigger:** Export request with revision_id + export_profile_id
- **Processing:**
  1. Compute deterministic key (revision + style_version + export_profile)
  2. Check artifact cache by deterministic key
  3. On miss: enqueue export job (idempotent)
  4. Worker re-validates constraints via ConstraintEngine (REQ-0003)
  5. Worker generates production output, stores in S3
- **Error cases:** Constraint drift detected (terminal, 409)

#### SB-04: Legacy Export Retrieval

- **Trigger:** Export request for a legacy design
- **Processing:**
  1. Check local artifact cache
  2. On hit: return cached artifact
  3. On miss: call third-party legacy service
  4. Store retrieved artifact in cache for future requests
- **Error cases:** Third-party unavailable and cache miss (503, retryable)
<!-- DA:END block:fsd.system_behaviour -->

### Api Contracts

<!-- DA:BEGIN block:fsd.api_contracts -->
#### Endpoints

| Method | Path | Purpose | Idempotency |
|--------|------|---------|-------------|
| POST | /designs/{design_id}/revisions | Create new decoration revision | Idempotency-Key header |
| GET | /designs/{design_id}/revisions/{revision_id} | Retrieve revision snapshot | N/A (read) |
| POST | /designs/{design_id}/revisions/{revision_id}/preview | Request preview generation | Idempotent by revision_id |
| GET | /artifacts/{artifact_id} | Retrieve generated artifact | N/A (read) |
| POST | /designs/{design_id}/revisions/{revision_id}/export | Request production export | Idempotent by deterministic key |
| GET | /legacy/designs/{legacy_ref}/export | Retrieve legacy export (read-through cache) | N/A (read) |

#### Common Headers

- `Authorization: Bearer <token>` -- token-based identity with role claims (ASM-0002)
- `Idempotency-Key: <uuid>` -- required on mutation endpoints (ADR-0003)

#### Error Response Format

All errors use RFC 7807 ProblemDetails with stable `error_code` field:

- `CONSTRAINT_VIOLATION` -- element violates decoration space rules (422)
- `STYLE_VERSION_NOT_FOUND` -- referenced style version does not exist (404)
- `REVISION_NOT_FOUND` -- referenced revision does not exist (404)
- `EXPORT_CONSTRAINT_DRIFT` -- export-time validation differs from save-time (409)
- `LEGACY_SERVICE_UNAVAILABLE` -- third-party service unreachable, cache miss (503)
- `IDEMPOTENCY_CONFLICT` -- same key used with different payload (409)
<!-- DA:END block:fsd.api_contracts -->

### Data Contracts

<!-- DA:BEGIN block:fsd.data_contracts -->
#### DecorationRevision (Persisted)

| Field | Type | Description |
|-------|------|-------------|
| revision_id | UUID | Primary key |
| design_id | UUID | Parent design reference |
| style_version_id | UUID | Pinned at save time (ADR-0006) |
| elements | JSON array | Collection of DecorationElement objects |
| constraint_result | JSON | ConstraintEngine validation output at save time |
| created_at | timestamp | Immutable creation time |
| idempotency_key | UUID | Unique key for duplicate detection |

#### DecorationElement (Embedded in Revision)

| Field | Type | Description |
|-------|------|-------------|
| element_id | UUID | Unique within revision |
| type | enum | `text` or `graphic` |
| space_id | string | Decoration space identifier |
| content | string/JSON | Text content or asset reference |
| position_x | decimal | X position within space |
| position_y | decimal | Y position within space |
| scale | decimal | Scale factor |
| rotation | decimal | Rotation in degrees |

#### Artifact (Persisted)

| Field | Type | Description |
|-------|------|-------------|
| artifact_id | UUID | Primary key |
| type | enum | `preview_overlay` or `production_export` |
| design_revision_id | UUID | Source revision |
| style_version_id | UUID | Style version used |
| export_profile_id | UUID (nullable) | Export profile (exports only) |
| deterministic_key | string | Hash for cache lookup (ADR-0004) |
| storage_ref | string | S3 object key |
| created_at | timestamp | Generation time |
<!-- DA:END block:fsd.data_contracts -->

