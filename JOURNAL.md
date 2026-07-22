# JOURNAL

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/68

**Issue title:** Add a safety event count to the health check endpoint

**Tier:** [x] Tier 1 [ ] Tier 2 [ ] Tier 3

**Problem summary:**
The `/health` endpoint already returns a `safety_events_last_hour` field in its response, but it's hardcoded to `0` (see `api/routes/health.py`) instead of reflecting real data. Separately, `safety/monitoring.py` already has a working `SafetyMonitor` class that logs safety events (PII detected, injection attempts, content filtered, etc.) into Redis and can return a count per event type via `get_event_count()` — but nothing in the health endpoint calls it. A successful fix wires the health check up to `SafetyMonitor` so it reports a real aggregate count across event types, and addresses the fact that the existing Redis counters use a 24-hour TTL rather than a true rolling one-hour window, so the count returned actually matches what "last hour" claims to mean. This affects the API layer (`api/routes/health.py`) and the safety monitoring module (`safety/monitoring.py`).

**Branch name:** fix/68-health-check-safety-event-count

**Setup confirmation:** [x] App runs locally at localhost:5173

**Selection notes (scope reasoning):**
- Labeled `tier-1` / `good first issue`, estimated 2–4 hours — appropriately scoped for a first issue in this codebase.
- Touches exactly two files with a clear, bounded change (wire an existing class into an existing endpoint), not a cross-cutting refactor.
- Note: another cohort member (RadRebelSam) commented on this issue two weeks ago saying they started work on branch `fix/68-health-check-safety-event-count`, and there's an open PR (#210) already linked to the issue. Proceeding anyway — duplication is acceptable here — but this is worth being aware of if the issue gets closed out from under this branch.

**Cohort ledger:** [ ] Issue added to cohort ledger
