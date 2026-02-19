# Bulk-First Post-Launch Failure (ABORTED)

**Status:** POST_LAUNCH_FAILURE_ABORT  
**Premium Budget Consumed:** 1 / 1

## Failure Reason

Bulk-first failed: Cross-phase bulk failed: Code: WU_TARGET_MISMATCH
Expected phase (raw): all
Observed phase (raw): discovery
Expected phase (canonical): all
Observed phase (canonical): discovery
Remediation: If observed is a known alias, add it to PhaseNameCanonicalizer. Otherwise fix provider objective or provider output.. Handoff written to docs/discovery/OPUS_HANDOFF.md

## Fail-Fast Behavior

The single Opus bulk-first invocation failed after provider launch. Per the fail-fast contract:
1. NO retries attempted for bulk-first (prevents wasting premium budget)
2. NO fallback to Haiku in the same run (fail-fast abort)
3. Run ABORTED to prevent cascading failures

## Diagnostics

- Provider failure evidence: `provider_failure.json not found`
- Run evidence directory: `.discovery/runs/<run_id>/provider/`
- Review `provider_failure.json` for failure kind, exit code, stderr/stdout tails

## Remediation

1. Review provider failure evidence to diagnose root cause
2. Fix the underlying issue (configuration, input data, provider bug, etc.)
3. Re-run `disco` or `da run --phase all`
4. Bulk-first will attempt again with the single Opus invocation

## Why Fail-Fast?

Continuing to per-phase execution after bulk-first failure:
- Would process work units individually with Haiku (expensive, slow)
- Might cascade the same failure across multiple phases
- Wastes time when the root cause needs fixing first

Aborting immediately allows fast iteration on the fix.

