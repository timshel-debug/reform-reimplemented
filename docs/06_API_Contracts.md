# API Contracts

All endpoints are internal to the platform; external/public exposure is out of scope. Authentication specifics depend on ASM-0002 (TODO). 

## Endpoints

### Styles/spaces (REQ-0001)
* `GET /styles/{styleVersionId}/spaces`

### Decoration persistence (REQ-0004/REQ-0005/REQ-0010)
* `GET /designs/{designId}/decoration`
* `POST /designs/{designId}/decoration/revisions` (supports `Idempotency-Key`)
* `POST /designs/{designId}/clone` (legacy-to-owned clone for editing; gated by routing policy)

### Preview jobs (REQ-0006)
* `POST /previews`
* `GET /previews/{previewJobId}`

### Export jobs (REQ-0007)
* `POST /exports`
* `GET /exports/{exportJobId}`

### Assets (REQ-0009)
* `GET /assets`
* `POST /admin/assets` / `PATCH /admin/assets/{assetId}` / `DELETE /admin/assets/{assetId}` (admin)

### Legacy exports (REQ-0008)
* `POST /legacy/exports`

### Rollout and routing controls (REQ-A-0001/REQ-A-0003)
* `GET /routing/policy/evaluate?designId=...&tenantId=...&styleVersionId=...&userId=...` (debug/admin)
* `PATCH /admin/routing/flags/{scope}` (audited flag changes; scope `env|tenant|style|user`)

### Legacy orderability coverage (REQ-A-0002)
* `POST /admin/legacy/coverage/backfill` (admin trigger for scheduled pre-generation workflow)
* `GET /legacy/coverage` (returns `% covered`, cohort filters, retention window)
* Scheduled interface: `legacy-coverage-backfill` worker schedule (documented internal job contract)

## Critical E2E flows

### Routing evaluation flow (rollout + rollback)

```mermaid
sequenceDiagram
  participant UI as Design Lab UI
  participant A as Decoration API
  participant R as Routing Policy Engine
  participant AUD as Audit Log

  UI->>A: Open/create design
  A->>R: Evaluate(env, tenant, style, user, designType)
  R-->>A: route + provider + editPolicy
  A-->>UI: Deterministic route decision

  Note over A,AUD: Admin flag changes are written with who/when/what
```

### Save and preview flow

```mermaid
sequenceDiagram
  participant UI as Design Lab UI
  participant P as Platform
  participant A as Decoration API
  participant Q as Queue
  participant W as Worker
  participant S3 as Object Storage
  
  UI->>P: Edit decoration
  P->>A: POST /designs/{id}/decoration/revisions
  A->>A: Validate constraints
  A-->>P: 201 {designRevisionId}
  
  P->>A: POST /previews
  A->>Q: Enqueue job
  A-->>P: 202 {previewJobId}
  
  W->>Q: Poll job
  W->>W: Render overlay
  W->>S3: Write artifact
  
  P->>A: GET /previews/{jobId}
  A-->>P: 200 {status, artifactUrl}
```

### Export flow with idempotency

```mermaid
sequenceDiagram
  participant P as Platform
  participant A as Decoration API
  participant Q as Queue
  participant W as Worker
  participant S3 as Object Storage
  
  P->>A: POST /exports<br/>Idempotency-Key: xyz
  A->>A: Check existing job
  alt Job exists
    A-->>P: 200 {exportJobId}
  else New job
    A->>Q: Enqueue job
    A-->>P: 202 {exportJobId}
  end
  
  W->>Q: Poll job
  W->>W: Validate constraints
  W->>W: Generate export
  W->>S3: Write artifact
  
  P->>A: GET /exports/{jobId}
  A-->>P: 200 {status, artifactUrl}
```

### Legacy export with cache

```mermaid
sequenceDiagram
  participant P as Platform
  participant A as Decoration API
  participant C as Artifact Cache
  participant L as Legacy Service
  participant S3 as Object Storage
  
  P->>A: POST /legacy/exports
  A->>C: Check cache
  alt Cache hit
    C-->>A: Artifact URL
    A-->>P: 200 {artifactUrl}
  else Cache miss
    A->>L: Request export
    L-->>A: Export data
    A->>S3: Store artifact
    A->>C: Update cache
    A-->>P: 200 {artifactUrl}
  end
```

### Legacy first-time order during third-party outage

```mermaid
sequenceDiagram
  participant P as Platform
  participant A as Decoration API
  participant C as Legacy Coverage Store
  participant S3 as Object Storage
  participant L as Legacy Service

  P->>A: Order requires legacy export
  A->>C: Check coverage/cached artifact
  alt Cached or pre-generated
    C-->>A: artifactUrl
    A-->>P: 200 {artifactUrl}
  else Not cached and third-party down
    A->>L: Request export
    L--xA: outage
    A-->>P: bounded failure mode + retryable policy response
  end
```

## Error codes (stable)
* `DECORATION_CONSTRAINT_VIOLATION` (400) — REQ-0003
* `STYLE_SPACE_NOT_FOUND` (404) — REQ-0001/REQ-0004
* `ASSET_NOT_FOUND` (404) — REQ-0009
* `FORBIDDEN` (403) — NFR-0004
* `ROUTING_POLICY_UNRESOLVED` (409) — REQ-A-0001/REQ-A-0003
* `LEGACY_ORDERABILITY_POLICY_UNMET` (503) — REQ-A-0002

ProblemDetails includes `error_code` + `correlation_id`.

Routing-relevant responses include `decoration_provider` (`legacy|owned`) and applicable `edit_policy` for deterministic UI behavior. 

Deterministic routing precedence is always `environment → tenant → style → user`, including clone/backfill-gated flows. 
