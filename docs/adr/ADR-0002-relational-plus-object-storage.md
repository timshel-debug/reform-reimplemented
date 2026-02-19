# ADR-0002 — Relational metadata store + object storage for blobs

## Status
Accepted

## Context
Must persist editable state and binary artifacts for months. 

## Decision
Relational DB for metadata + object storage for assets/artifacts.

## Consequences
* Efficient queries and traceability.
* Retention enforced by lifecycle policies (ASM-0008 TODO). 
