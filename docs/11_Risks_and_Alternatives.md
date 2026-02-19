# Risks and Alternatives

## Risks

* Legacy first-time export during third-party outage (OQ-0001/OQ-0008). 
* Rollout misrouting risk if policy order is not deterministic or not auditable (`env → tenant → style → user`). 
* Export format ambiguity (OQ-0005). 
* Determinism definition gaps (ASM-0005). 
* Constraint complexity unknown (ASM-0007). 
* Fonts/assets licensing (ASM-0006). 
* Style version coupling (CON-0003). 
* Prototype may fail to prove full thin-slice feasibility in planned timeline, delaying go/no-go confidence. 

## Alternatives

* Full migration of legacy designs (candidate for REQ-A-0002 compliance; decision pending OQ-A-0001). 
* Pre-generate legacy exports for active cohorts as outage fallback (candidate for REQ-A-0002 compliance). 
* Legacy edit modes: `legacy-only` vs `clone-to-new` vs editable import (decision pending OQ-A-0002). 
* Synchronous preview/export (rejected due to NFR-0001). 
* Snapshot space definitions into each revision (deferred; OQ-0006). 
