# ADR-0005 — Legacy continuity via dual-run + read-through cache

## Status
Accepted

## Context
Legacy designs must remain orderable for retention window and rely on immutable third-party service. 

## Decision
Dual-run for legacy export + store retrieved exports in first-party object storage for reuse.

## Consequences
* Reduces repeat third-party dependency.
* Does not cover first-time export during outage; policy TBD (OQ-0001/OQ-0008). 
