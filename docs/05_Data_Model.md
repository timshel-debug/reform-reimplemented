# Data Model

Relational metadata + object storage blobs (ADR-0002). Retention is configurable (ASM-0008 TODO). 

## Tables (logical)

* `designs` — design ownership + provider metadata (`decoration_provider=owned|legacy`) + legacy flags.
* `style_versions_cache` — cached style/space definitions (REQ-0001).
* `design_revisions` — immutable snapshots (REQ-0004/REQ-0010).
* `revision_elements` — per-space elements.
* `assets` — artwork library metadata (REQ-0009).
* `jobs_preview`, `jobs_export` — async job tracking (REQ-0006/REQ-0007).
* `artifacts` — stored previews/exports and trace metadata (REQ-0010).
* `routing_flags` — rollout/cutover/rollback flag values by scope (`env|tenant|style|user`) and evaluation priority. 
* `routing_flag_audit` — auditable change history (`who/when/what`) for rollout and rollback controls. 
* `legacy_coverage` — coverage strategy status for legacy orderability (`cached_export|migrated|alternate_policy`). 

## Object storage keys (logical)

* `assets/{asset_id}/{content_hash}`
* `previews/{design_revision_id}/{side_id}/{preview_profile_id}/{content_hash}.png`
* `exports/{design_revision_id}/{export_profile_id}/{content_hash}.pdf`
* `legacy/{provider}/{legacy_reference}/{content_hash}.pdf`

## Entity-relationship diagram

```mermaid
erDiagram
  DESIGNS ||--o{ DESIGN_REVISIONS : has
  DESIGNS {
    uuid design_id PK
    uuid style_version_id FK
    string decoration_provider
    string legacy_provider
    string legacy_reference
  }
  DESIGN_REVISIONS ||--o{ REVISION_ELEMENTS : contains
  DESIGN_REVISIONS {
    uuid design_revision_id PK
    uuid design_id FK
    timestamp created_at
    jsonb metadata
  }
  REVISION_ELEMENTS }o--|| ASSETS : references
  REVISION_ELEMENTS {
    uuid element_id PK
    uuid design_revision_id FK
    string type
    uuid space_id
    jsonb transform
    jsonb content
    uuid asset_id FK
  }
  ASSETS {
    uuid asset_id PK
    string content_hash
    jsonb availability_rules
    string url
  }
  DESIGN_REVISIONS ||--o{ JOBS_PREVIEW : generates
  JOBS_PREVIEW {
    uuid job_id PK
    uuid design_revision_id FK
    string status
    string idempotency_key
  }
  DESIGN_REVISIONS ||--o{ JOBS_EXPORT : generates
  JOBS_EXPORT {
    uuid job_id PK
    uuid design_revision_id FK
    string status
    string idempotency_key
  }
  JOBS_PREVIEW ||--o| ARTIFACTS : produces
  JOBS_EXPORT ||--o| ARTIFACTS : produces
  ARTIFACTS {
    uuid artifact_id PK
    uuid job_id FK
    string type
    string url
    string content_hash
    jsonb trace_metadata
  }
  STYLE_VERSIONS_CACHE {
    uuid style_version_id PK
    jsonb spaces_definition
    timestamp synced_at
  }
  ROUTING_FLAGS {
    uuid routing_flag_id PK
    string scope_type
    string scope_id
    string route_mode
    boolean enabled
    timestamp updated_at
  }
  ROUTING_FLAG_AUDIT {
    uuid audit_id PK
    uuid routing_flag_id FK
    string changed_by
    timestamp changed_at
    jsonb change_payload
  }
  LEGACY_COVERAGE {
    uuid design_id PK
    string coverage_state
    timestamp evaluated_at
    timestamp artifact_expires_at
  }
  ROUTING_FLAGS ||--o{ ROUTING_FLAG_AUDIT : records
  DESIGNS ||--o| LEGACY_COVERAGE : tracked_by
```
