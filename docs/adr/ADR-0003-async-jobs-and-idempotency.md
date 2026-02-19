# ADR-0003 — Async preview/export jobs with idempotent keys

## Status
Accepted

## Context
High save volumes and edit cycles require scalable preview/export. 

## Decision
Use async job endpoints with deterministic job keys for dedupe.

## Consequences
* Requires job status and artifact serving endpoints.
* UI polls or subscribes (mechanism TODO via OQ-0007). 
