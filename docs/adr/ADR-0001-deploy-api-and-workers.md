# ADR-0001 — Deploy as Decoration API + scale-out workers

## Status
Accepted

## Context
Preview and export workloads must support horizontal scaling and should not block interactive editing. 

## Decision
Deploy:
* A stateless **Decoration API** for synchronous validation/persistence and job submission.
* Separate **Render/Export Worker** processes for preview and export generation.

## Consequences
* Independent scaling of API vs compute-heavy workers.
* Async job APIs required (ADR-0003).
