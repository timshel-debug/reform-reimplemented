# ADR-0006 — Pin style version at save; compatibility mapping for reload

## Status
Accepted

## Context
Styles/spaces are versioned; saved designs must remain compatible. 

## Decision
Pin `style_version_id` per revision; allow explicit mapping to newer versions if required.

## Consequences
* Supports AC-REQ-0001-02.
* Mapping rules/tooling are TODO (OQ-0006). 
