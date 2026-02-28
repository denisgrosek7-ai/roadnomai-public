# VAL 6.2 Fire Drill Runbook (10 min, 3 terminals)

Quick sanity variant:
- `docs/val6_2_fire_drill_quick_run.md`

Assumptions:
- Backend runs on `http://localhost:8080`
- Endpoint is `POST /v1/trip/generate`
- Chaos toggle is implemented (`CHAOS_ENABLED`, `CHAOS_PROVIDER_MODE`, ...)
- Logs are JSON to stdout

## Terminal 1 — Backend lifecycle (Warmup -> Chaos -> Recovery)

Always run from backend directory so `backend/.env` is loaded:

```bash
cd backend
```

Warmup (0-2 min):

```bash
export PORT=8080
export NODE_ENV=development
export CB_STORAGE=redis
export CB_ENABLED=true

export CHAOS_ENABLED=0
export CHAOS_PROVIDER_FAIL_RATE=0.4
export CHAOS_PROVIDER_MODE=rate_limit
export CHAOS_PROVIDER_TIMEOUT_MS=5000
export CHAOS_RETRY_AFTER_MS=2000

node server.js 2>&1 | tee /tmp/rn-backend.log
```

Chaos phase (2-7 min), restart backend:

```bash
# Ctrl+C previous process, then:
cd backend
export PORT=8080
export NODE_ENV=development
export CB_STORAGE=redis
export CB_ENABLED=true

export CHAOS_ENABLED=1
export CHAOS_PROVIDER_FAIL_RATE=0.4
export CHAOS_PROVIDER_MODE=rate_limit
export CHAOS_RETRY_AFTER_MS=2000

node server.js 2>&1 | tee /tmp/rn-backend.log
```

Recovery phase (7-10 min), restart backend:

```bash
# Ctrl+C previous process, then:
cd backend
export PORT=8080
export NODE_ENV=development
export CB_STORAGE=redis
export CB_ENABLED=true

export CHAOS_ENABLED=0

node server.js 2>&1 | tee /tmp/rn-backend.log
```

Optional second run (timeout chaos):

```bash
export CHAOS_ENABLED=1
export CHAOS_PROVIDER_FAIL_RATE=0.4
export CHAOS_PROVIDER_MODE=timeout
export CHAOS_PROVIDER_TIMEOUT_MS=5000
```

## Terminal 2 — Heartbeat / telemetry correlator

```bash
cd <repo-root>
./tools/chaos_heartbeat.sh /tmp/rn-backend.log
```

Expected signals:
- Warmup: `cache_hit` rises, `provider_call_success` present
- Chaos: `cb_state_change` -> OPEN, `cb_call_blocked` rises, provider calls drop
- Recovery: HALF_OPEN trials then CLOSED

## Terminal 3 — Load runner (dual-lane)

```bash
cd <repo-root>
BACKEND_SHARED_TOKEN=YOUR_TOKEN \
CHAOS_TEST_DURATION_MS=600000 \
CHAOS_REPEAT_RPS=35 \
CHAOS_UNIQUE_RPS=15 \
node tools/chaos_load_test.js http://localhost:8080
```

## Post-run evidence

1) Keep the script `SUMMARY JSON` and PASS/FAIL table.

2) Event counts:

```bash
rg -N '"event":"cb_state_change"' /tmp/rn-backend.log | wc -l
rg -N '"event":"cb_call_blocked"' /tmp/rn-backend.log | wc -l
rg -N '"event":"provider_call_success"' /tmp/rn-backend.log | wc -l
rg -N '"event":"provider_call_failure"' /tmp/rn-backend.log | wc -l
rg -N '"event":"waiter_timeout"' /tmp/rn-backend.log | wc -l
```

3) RFC-001 degraded-mode shape quick spot check:

```bash
rg -N '"error":\{"code":"E_DEGRADED_MODE"' /tmp/rn-backend.log | head
```

## Quick pass/fail targets

- Repeat lane warmup P95 <= 100ms (goal)
- Unique lane warmup P95 <= 800ms (target)
- Chaos: fail-fast behavior acceptable, RFC-001 envelope preserved
- Stability: no crash loops, no runaway waiter timeouts
