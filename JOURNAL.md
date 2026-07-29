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

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** [link to this commit — fill in after pushing]

**Reproduction summary:**
Started the backend locally (`uvicorn api.main:app --reload --host 0.0.0.0 --port 8000`) with `db`/`redis`/`vector-db` up via `docker compose up -d`, then sent a `GET http://localhost:8000/health` request via Postman. The response came back `503 Service Unavailable` with `"safety_events_last_hour": 0` in the body — confirmed by reading [api/routes/health.py](api/routes/health.py#L78), the field is hardcoded to `0` and never calls into `SafetyMonitor` (in [safety/monitoring.py](safety/monitoring.py)), which already has a working `log_event()`/`get_event_count()` API but isn't wired into the health check at all.

**PLAN.md link:** [PLAN.md](https://github.com/galipcagan/pathreview/blob/fix/68-health-check-safety-event-count/PLAN.md)

**Walkthrough video (recommended):** (not recorded)

**Blockers or open questions:**
- While reproducing, the `/health` endpoint also misreported `postgres` and `redis` as `"unhealthy"` even though both containers were confirmed healthy — two separate pre-existing bugs (a raw-SQL string passed where SQLAlchemy requires `text("SELECT 1")`, and a reference to a `Settings.redis_host` attribute that doesn't exist). Both are out of scope for this PR and not something I'm fixing here, but noting them since they're in the same file/function.
- Also found: `SafetyMonitor.get_event_count()` accepts a `window_hours` parameter but never actually uses it — the Redis counter it reads just has a flat 24h TTL, not a real rolling window. Need to decide in the fix whether "last hour" should be a true rolling window (sorted-set based, mirroring `safety/rate_limiter.py`'s `RateLimiter` pattern) or a simpler hour-bucketed counter — see PLAN.md.
