# VAL 6.2 Quick Run (2 min, 3 terminals)

Purpose:
- Validate wiring before full 10-min drill
- Ensure chaos toggle, CB transitions, Abort budget, and RFC-001 error shape are working

Profile:
- Warmup: 30s (chaos off)
- Chaos: 60s (chaos on)
- Recovery: 30s (chaos off)
- Total load: 20 RPS (repeat 14, unique 6)

## Terminal 1 — Backend

Warmup:

```bash
cd backend
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

Chaos (after 30s):

```bash
# Ctrl+C, then:
cd backend
export PORT=8080
export NODE_ENV=development
export CB_STORAGE=redis
export CB_ENABLED=true
export CHAOS_ENABLED=1
export CHAOS_PROVIDER_FAIL_RATE=0.4
export CHAOS_PROVIDER_MODE=rate_limit
node server.js 2>&1 | tee /tmp/rn-backend.log
```

Recovery (after 90s total):

```bash
# Ctrl+C, then:
cd backend
export PORT=8080
export NODE_ENV=development
export CB_STORAGE=redis
export CB_ENABLED=true
export CHAOS_ENABLED=0
node server.js 2>&1 | tee /tmp/rn-backend.log
```

## Terminal 2 — Heartbeat

```bash
cd <repo-root>
./tools/chaos_heartbeat.sh /tmp/rn-backend.log
```

Expected in chaos:
- `cb_state_change` to OPEN
- `cb_call_blocked` rises
- `provider_call_success/failure` drops

## Terminal 3 — Quick load

```bash
cd <repo-root>
BACKEND_SHARED_TOKEN=YOUR_TOKEN \
node tools/chaos_load_test.js http://localhost:8080 --quick
```

Equivalent explicit flags:

```bash
BACKEND_SHARED_TOKEN=YOUR_TOKEN \
node tools/chaos_load_test.js http://localhost:8080 \
  --duration=120000 --warmup=30000 --chaos=60000 --recovery=30000 \
  --rps=20 --repeat-rps=14 --unique-rps=6
```

## Quick PASS criteria

- Script completes and prints per-lane summaries
- During chaos, CB opens at least once (`cb_state_change`)
- `cb_call_blocked` appears
- Non-2xx responses remain RFC-001 envelopes (no HTML errors)
- No crash loop
