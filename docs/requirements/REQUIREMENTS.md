# Requirements

## How to use
- Each requirement has a unique ID (REQ-#### pattern).
- Reserved ranges for multi-agent system:
  - REQ-1xxx: Orchestrator
  - REQ-2xxx: Disco (Discovery Agent)
  - REQ-3xxx: Imp (Implementation Agent)
  - REQ-4xxx: Verify
  - REQ-5xxx: Audit
  - REQ-9xxx: Shared guardrails

## Requirements

### REQ-2000: REQ-2000 - ASM-0008 — Retention window policy (PARTIAL/TODO)

**Source:** [docs/requirements/sources/REQ-2000-auto-derived.md](docs/requirements/sources/REQ-2000-auto-derived.md#L6-L9)

**Statement:**

TODO: confirm exact window and whether “export artifacts must be stored” vs “inputs must be stored and re-rendered”.

**Acceptance Criteria:**
- AC1: Verify: REQ-2000 - ASM-0008 — Retention window policy (PARTIAL/TODO)

**Disco Traceability:** REQ-2000

---

### REQ-2001: REQ-2001 - ASM-0009 — Existing platform design identity (TODO)

**Source:** [docs/requirements/sources/REQ-2000-auto-derived.md](docs/requirements/sources/REQ-2000-auto-derived.md#L10-L13)

**Statement:**

Risk/Impact: if platform uses per-cart/session identifiers, decoration data modeling must change.

**Acceptance Criteria:**
- AC1: Verify: REQ-2001 - ASM-0009 — Existing platform design identity (TODO)

**Disco Traceability:** REQ-2001

---

### REQ-2002: REQ-2002 - OQ-0009 — Third-party legacy export API capabilities (TODO)

**Source:** [docs/requirements/sources/REQ-2000-auto-derived.md](docs/requirements/sources/REQ-2000-auto-derived.md#L14-L17)

**Statement:**

Risk/Impact: if third-party export is not reliable, platform must pre-cache exports earlier or enforce cutoff policy.

**Acceptance Criteria:**
- AC1: Verify: REQ-2002 - OQ-0009 — Third-party legacy export API capabilities (TODO)

**Disco Traceability:** REQ-2002

---

### REQ-2003: REQ-2003 - DecorationRevision (immutable)

**Source:** [docs/requirements/sources/REQ-2000-auto-derived.md](docs/requirements/sources/REQ-2000-auto-derived.md#L18-L21)

**Statement:**

Must satisfy constraint engine at save time.

**Acceptance Criteria:**
- AC1: Verify: REQ-2003 - DecorationRevision (immutable)

**Disco Traceability:** REQ-2003

---

### REQ-2004: REQ-2004 - Context

**Source:** [docs/requirements/sources/REQ-2000-auto-derived.md](docs/requirements/sources/REQ-2000-auto-derived.md#L22-L26)

**Statement:**

Styles/spaces are versioned; saved designs must remain compatible.

**Acceptance Criteria:**
- AC1: Verify: REQ-2004 - Context

**Disco Traceability:** REQ-2004

---

