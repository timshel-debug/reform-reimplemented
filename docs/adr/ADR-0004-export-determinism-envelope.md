# ADR-0004 — Deterministic export identity uses revision + profile

## Status
Accepted (pending ASM-0005)

## Context
Exports must be reproducible for a saved design state. 

## Decision
Export identity = `design_revision_id` + `style_version_id` + `export_profile_id`.

## Consequences
* Requires versioned assets/fonts to guarantee byte-identical exports (ASM-0006 TODO). 
