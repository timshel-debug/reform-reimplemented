# Domain Model

## Key aggregates

### DecorationDesign
* Identity: `design_id` (platform identifier; ASM-0009 TODO). 
* Attributes: `style_version_id`, `decoration_provider` (`owned|legacy`), optional legacy provider/reference. 

### DecorationRevision (immutable)
* Identity: `design_revision_id`
* Contains per-space `DecorationElement` collections.
* Must satisfy constraint engine at save time. 

### DecorationElement
* Type: `text` or `graphic` (REQ-0002). 
* Stores transform (position/scale; rotation optional OQ-0003). 

### Asset
* Approved artwork selectable in editor (REQ-0009). 

### Artifact
* Preview overlays and production exports; stamped with revision + style version for traceability (REQ-0010). 

## Domain service: ConstraintEngine
Single authoritative validation used by both editor-time save and export-time generation to prevent drift (REQ-0003). 

## Domain model diagram

```mermaid
classDiagram
  class DecorationDesign {
    +design_id
    +style_version_id
    +decoration_provider
    +legacy_provider?
    +legacy_reference?
  }
  class DecorationRevision {
    +design_revision_id
    +created_at
    +validate()
  }
  class DecorationElement {
    +element_id
    +type: text|graphic
    +space_id
    +transform
    +content
  }
  class Asset {
    +asset_id
    +content_hash
    +availability_rules
  }
  class Artifact {
    +artifact_id
    +type: preview|export
    +revision_id
    +style_version_id
    +url
  }
  class ConstraintEngine {
    +validate(revision, spaces)
  }
  
  DecorationDesign "1" --> "*" DecorationRevision
  DecorationRevision "1" --> "*" DecorationElement
  DecorationElement "*" --> "0..1" Asset
  DecorationRevision "1" --> "*" Artifact
  ConstraintEngine ..> DecorationRevision : validates
```
