# reform-reimplemented — Solution Design Bundle (DSDS)

This repository is a **documentation-first solution design** for an owned *Decoration Platform* that replaces the current third‑party decoration service for **new designs**, while preserving **legacy design continuity** during a controlled transition.

## What problem this solves
- Provide first‑party ownership of **decoration editing**, **decoration data persistence**, **preview rendering**, and **production export generation**.
- Reduce operational risk from third‑party dependency for *new* designs.
- Support an explicit **dual‑stream transition** (legacy + owned) with defined rollout controls and legacy orderability policies.

## Start here
1. **Inputs / source-of-truth**
   - `inputs/SPEC-DECOR-0001.md` (base requirements)
   - `inputs/SPEC-DECOR-0001-ADDENDUM-A.md` (rollout/legacy/prototyping/agentic addendum)
   - Index: `docs/00_Source_Index.md`

2. **Core DSDS docs**
   - `docs/01_Overview.md` — context + goal + workflows
   - `docs/02_Requirements.md` — consolidated REQ/NFR/CON + acceptance criteria
   - `docs/03_Architecture.md` — containers, flows, patterns
   - `docs/04_Domain_Model.md` / `docs/05_Data_Model.md` — domain + persistence
   - `docs/06_API_Contracts.md` — API surface + sequencing/idempotency
   - `docs/07_Security_and_Trust_Boundaries.md` — authn/authz + trust boundaries
   - `docs/08_Operations.md` — deployment/ops on AWS, scaling, observability
   - `docs/11_Risks_and_Alternatives.md` — key risks + tradeoffs

3. **Decision records**
   - `docs/adr/` — ADR-0001..ADR-0008 covering deployment topology, storage strategy, async jobs/idempotency, export determinism, legacy dual-run, style pinning, and legacy policies.

4. **Rendered copies**
   - `docs/rendered/` and `docs/rendered-attached/` contain HTML renders of the markdown for easy sharing.

## Implementation slices
The recommended delivery sequence is captured in:
- `docs/10_Implementation_Plan.md` — human-readable slice ordering (SLICE-0001+)
- `docs/backlog/` — deterministic execution backlog:
  - `docs/backlog/backlog.json` — machine-readable work items, deps, and scope guards
  - `docs/backlog/slices/SLICE-###.md` — one file per slice with requirements + acceptance criteria

## Discovery Agent & deterministic backlog (how it was produced)
This bundle was generated using a **Discovery Agent (“Disco”)** workflow to produce an auditable, repeatable backlog from the input specs.

Disco was used to:
- Parse the input specs into structured requirements and traceability artifacts:
  - `docs/requirements/REQUIREMENTS.md`
  - `docs/requirements/sources/REQ-2000-auto-derived.md`
  - `docs/traceability/trace_index.json`
- Produce managed discovery outputs (with stable `DA:BEGIN/DA:END` blocks) that keep the narrative docs consistent:
  - `docs/discovery/*` (e.g., `HANDOFF.md`, `CORRECTNESS_CONTRACT.md`, candidates triage list)
- Generate a **well-formatted deterministic backlog**:
  - Stable slice IDs (`SLICE-###`)
  - Explicit dependency ordering
  - Explicit path scope allowlists per slice (to keep changes contained)
  - Standardized acceptance criteria (build/test gates + requirement-specific verification)
  - Clear requirement → slice traceability

Evidence of the Discovery Agent runs is retained under:
- `.discovery/registry/` — requirement registry outputs
- `.discovery/runs/` — run manifests, events, and provider context packs (for auditability/repro)
- `.discovery/block_hashes.json` — content-hash tracking for deterministic block regeneration

## Using this bundle with Copilot / implementation agents
- See `.github/copilot-instructions.md` for project-specific guidance and conventions.
- Execute work in slice order; keep the UI surface unified via a provider router (legacy vs owned) and drive rollout via deterministic policy (env → tenant → style → user).
- Keep “correctness” explicit:
  - Review `docs/discovery/CORRECTNESS_CONTRACT.md`
  - Update build/test commands once code exists (this bundle is design-first).

## Notes
- This is a **solution design** bundle; it does not include full implementation code.
- Several items remain intentionally open (SLOs, exact export format, retention policy). Track these in:
  - `docs/00_Assumptions_and_OpenQuestions.md`
