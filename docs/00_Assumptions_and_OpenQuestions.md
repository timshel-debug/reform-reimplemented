# Assumptions and Open Questions

## Assumptions (ASM)

> Status is copied from the requirement specification unless otherwise stated. 

### ASM-0001 — Deployment target is AWS (Confirmed)
* Decision impact: all proposed deployment components are expressed in AWS primitives. 

### ASM-0002 — Platform provides authenticated identity and role claims (TODO)
* TODO: confirm identity mechanism, token format, and where “Customer vs Admin/Operations” is enforced.
* Risk/Impact: auth boundary and audit trails cannot be finalized; API contracts may need claim field changes. 

### ASM-0003 — Style/spaces are readable from platform sync (Answered)
* Assumption used: Decoration Platform can read a platform-owned “Style Read Model” via API or event feed. 

### ASM-0004 — Colouring pipeline exists upstream (Answered)
* Assumption used: preview compositing overlays decoration outputs on top of an upstream garment base image per side. 

### ASM-0005 — Export determinism semantics (TODO)
* Best-effort default: export is deterministic for the same `design_revision_id` + `style_version_id` + `export_profile_id` (new concept; see ADR-0004) and produces the same bytes unless fonts/assets change.
* TODO: confirm acceptable sources of nondeterminism (font rendering versions, PDF metadata, timestamps).
* Risk/Impact: without a deterministic definition, regression testing and “idempotent export” acceptance (AC-REQ-0007-02) may not be provable. 

### ASM-0006 — Fonts/assets licensing (TODO)
* TODO: confirm licensing constraints for server-side rendering and embedding fonts/art in exports.
* Risk/Impact: export pipeline may be restricted to a curated font set; asset ingestion may require legal workflow. 

### ASM-0007 — Constraints are rule-expressible (PARTIAL/TODO)
* Best-effort default: constraints are expressed as data-driven rules evaluated in editor and export.
* Risk/Impact: if constraints require complex validation (e.g., stitch density simulation), the MVP architecture may need specialized compute and domain modeling. 

### ASM-0008 — Retention window policy (PARTIAL/TODO)
* Best-effort default: configurable retention, default 12 months.
* TODO: confirm exact window and whether “export artifacts must be stored” vs “inputs must be stored and re-rendered”.
* Risk/Impact: storage cost vs outage mitigation trade-offs (especially for legacy). 

### ASM-0009 — Existing platform design identity (TODO)
* Best-effort default: platform has a stable `design_id` for “full design” save/load; Decoration Platform stores decoration revisions keyed by that identifier.
* Risk/Impact: if platform uses per-cart/session identifiers, decoration data modeling must change. 

### ASM-0010 — Multi-tenant boundary (TODO)
* Best-effort default: designs are isolated by `account_id` (tenant) + `user_id` (owner) as provided by platform claims.
* Risk/Impact: data access patterns and authorization rules change if tenancy is not present. 

## Open Questions (OQ)

Open questions are copied and extended only where necessary to complete a design plan. 

### OQ-0001 — Legacy strategy
* TODO: decide between (a) migrate, (b) dual-run, (c) cutoff. 

### OQ-0002 — Legacy edit support
* TODO: define whether legacy designs open in legacy editor vs are converted and editable in first-party editor. 

### OQ-0003 — MVP feature set beyond text/graphics
* TODO: define support for layers, rotation, multi-colour embroidery rules, typography features. 

### OQ-0004 — Performance targets
* TODO: define preview latency SLOs, save latency SLOs, and concurrency. 

### OQ-0005 — Export format requirements
* TODO: confirm export format (PDF vs other), layout rules, per-space outputs, and factory integration. 

### OQ-0006 — Space definition snapshotting
* TODO: decide between snapshotting into design revision vs reference + mapping. 

### OQ-0007 — Preview API mode
* TODO: decide on synchronous preview vs async generation with caching. 

### OQ-0008 — Export timing strategy
* TODO: decide on order-time export vs pre-generation and caching. 

### OQ-0009 — Third-party legacy export API capabilities (TODO)
* Best-effort assumption: third-party can produce production exports for legacy designs at order time.
* TODO: confirm API endpoints, auth, rate limits, and SLAs.
* Risk/Impact: if third-party export is not reliable, platform must pre-cache exports earlier or enforce cutoff policy. 

## Addendum Open Questions (OQ-A)

These questions are mandatory to answer before design exit. 

### OQ-A-0001 — Legacy orderability approach
* TODO: choose primary strategy (`pre-generate exports` vs `migrate` vs `alternate retained-artifact policy`) and fallback order.
* Exit evidence required: passing outage scenario where uncached legacy design order still completes. 

#### Status
* OPEN — pending stakeholder signoff.

#### Recommended default (PROPOSED)
* Scheduled pre-generation for in-scope legacy cohorts + cached artifact read-through on demand.
* Keep migration as a deferred alternative, not MVP-critical.

#### Rationale
* Directly closes the known read-through-only gap (first-time legacy export during third-party outage).
* Minimizes scope expansion by extending current dual-run/cache model instead of introducing full migration in MVP.
* Provides measurable coverage governance (`% legacy cached`, uncached-age KPI) for go/no-go controls.

#### Validation / exit criteria
* Simulated third-party outage test passes for legacy order attempts when coverage policy is satisfied.
* Coverage KPI and max-age KPI are instrumented with alerts and weekly reporting.
* Retention policy for legacy cached artifacts is explicitly confirmed and enforced.

#### Decision owner
* Product + Operations + Architecture triad (final signoff authority).

### OQ-A-0002 — Explicit legacy edit policy
* TODO: choose one deterministic policy (`editable in owned`, `legacy-only`, `order-only + clone-to-new`) and define UI/API routing behavior.
* Exit evidence required: metadata/API alignment with `decoration_provider=legacy|owned` and route decision tests. 

#### Status
* OPEN — pending stakeholder signoff.

#### Recommended default (PROPOSED)
* Legacy designs remain order/export capable on legacy path.
* Editing uses `clone-to-new`: create an owned design clone, then edit in first-party editor.

#### Rationale
* Preserves continuity for legacy ordering while avoiding risky in-place legacy editing semantics.
* Aligns with owned-domain immutability/versioning patterns and deterministic provider routing.
* Constrains MVP complexity while still enabling customer editing via explicit conversion path.

#### Validation / exit criteria
* `POST /designs/{designId}/clone` (or equivalent) creates an owned design with editable state.
* Clone fidelity checks pass for required visual baseline and exportability.
* Routing and metadata (`decoration_provider=legacy|owned`, `edit_policy`) are deterministic in UI/API.

#### Decision owner
* Product + UX + Architecture triad (final signoff authority).

### OQ-A-0003 — Prototype exit criteria and signoff
* TODO: define measurable exit checklist, target timeline, and named approvers for go/no-go decision.
* Exit evidence required: completed thin-slice path (`place → persist → preview → export`) and documented risk mitigations. 

### OQ-A-0004 — Retention window confirmation
* TODO: confirm exact retention window and enforcement policy for legacy artifacts (minimum guidance >= 12 months).
* Exit evidence required: documented retention duration and lifecycle controls reflected in operations policy. 
