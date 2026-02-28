## Repository Scope (Public)

This repository contains **documentation and schemas only** (no proprietary engine source code).

### Included
- Documentation (`docs/`)
- RFC-001 schemas (`schemas/`)
- Release notes and baselines (`CHANGELOG.md`, `artifacts/`)
- Governance docs (`SECURITY.md`, `CONTRIBUTING.md`, `LICENSE`)

### Not Included
- Core orchestration engine implementation
- Provider adapters, prompts, and internal routing source code

The production engine is proprietary and remains in a private repository.

---

# roadnomai.app

A new Flutter project.

## Secure API Setup

Sensitive provider keys are now backend-only.

### 1) Configure backend proxy

```bash
cd backend
cp .env.example .env
# fill BACKEND_SHARED_TOKEN, GOOGLE_MAPS_SERVER_API_KEY, GEMINI_API_KEY, OPENAI_API_KEY
npm install
npm start
```

Backend now also serves transfer metadata at:

`GET /v1/transfer-info?fromLat=..&fromLng=..&toLat=..&toLng=..`

### 2) Configure Flutter app

```bash
cp .env.example .env
# fill ROADNOMAI_BACKEND_TOKEN
```

API base URL now supports `--dart-define`:

```bash
flutter run --dart-define=API_BASE_URL=http://localhost:8787
```

iOS simulator cannot use `localhost` to reach your Mac host process. Use your Mac LAN IP or hostname:

```bash
flutter run --dart-define=API_BASE_URL=http://<YOUR_MAC_IP>:8787
# or
flutter run --dart-define=API_BASE_URL=http://<your-hostname>.local:8787
```

Production example:

```bash
flutter run --release --dart-define=API_BASE_URL=https://api.your-domain.com
```

For iOS Google Maps SDK key, keep using `ios/Flutter/Secrets.xcconfig`.

## Transfer Hint Check

After `backend` and Flutter app are running, create a Search A plan with island hops (example: Philippines, Manila -> Negros -> Dumaguete).  
Between destination cards you should see transfer hints such as `Drive`, `Ferry`, `Flight`, or combined labels like `Ferry • Flight`.

## One-command Local Dev

From repo root:

```bash
npm run dev
```

This script starts backend (if not already running), waits for health check on `http://localhost:8787/healthz`, and then runs Flutter on `iPhone 16e`.

Optional device override:

```bash
ROADNOMAI_DEVICE="iPhone 16e" npm run dev
```

## Secret Safety Guard

Install repository git hooks once:

```bash
./scripts/install_git_hooks.sh
```

This enables `.githooks/pre-commit`, which blocks staged commits containing obvious API-key patterns.

## Plugin Sandbox Rules

Category plugins must live under `lib/registry/plugins/`.

Plugins are pure Dart extension points and must not import IO/network/internal API clients:

- `dart:io`, `dart:html`, `dart:js`, `dart:indexed_db`
- `package:http*`, `package:dio*`, `package:web_socket_channel*`
- internal proxy/client imports such as `secure_backend_proxy`

CI enforces this with custom lint rule `no_plugin_io_imports`.

## Handshake Manifest (VAL 4.1)

### Manifest endpoint

- Path: `GET /v1/manifest`
- Response contract (exact fields):
  - `min_supported_app_version` (`string`, semver)
  - `recommended_app_version` (`string`, semver)
  - `cache_epoch` (`int`)
  - `active_categories` (`array` of `{ id: string, enabled: bool }`)
  - `emergency_message` (`string | null`)

Example:

```json
{
  "min_supported_app_version": "1.0.0",
  "recommended_app_version": "1.2.0",
  "cache_epoch": 1,
  "active_categories": [
    { "id": "surprise_me", "enabled": true },
    { "id": "crystal_waters", "enabled": true },
    { "id": "hiking_beta", "enabled": false }
  ],
  "emergency_message": null
}
```

### Client behavior (HandshakeService)

- On startup:
  - fetch manifest from backend
  - persist manifest and timestamp in SharedPreferences:
    - `handshake.manifest.v1`
    - `handshake.last_fetch_ms.v1`
  - if fetch fails: load cached manifest and set `offlineMode=true`
- Version enforcement:
  - hard block when `appVersion < min_supported_app_version`
  - soft update prompt when `appVersion < recommended_app_version`

### Cache epoch (remote cache purge)

- `SignatureService` key material includes `epoch=<cache_epoch>`.
- Changing `cache_epoch` on backend invalidates all existing client cache entries safely (no schema changes required).

### Category availability rules

- `categoryStateFor(id)`:
  - `visible=true` only when category `id` exists in `active_categories` (when manifest is loaded)
  - `enabled` follows `active_categories[].enabled`
- Fallback when manifest is not loaded (first launch/offline):
  - safe UX fallback: locally known categories remain visible and enabled to avoid blocking startup.

### Emergency messaging

- If `emergency_message != null`, app shows maintenance overlay at root.

## VAL 4.2 — Per-Category Schema Versioning (Versioning Shield)

### Manifest contract (`active_categories` migration)

- Endpoint: `GET /v1/manifest`
- `active_categories` supports both formats (backward-compatible):

Legacy (VAL 4.1):

```json
"active_categories": ["surprise_me", "crystal_waters"]
```

VAL 4.2+:

```json
"active_categories": [
  { "id": "surprise_me", "enabled": true, "schema_version": 1 },
  { "id": "crystal_waters", "enabled": true, "schema_version": 2 }
]
```

Parsing rules:

- `String` element => `enabled=true`, `schema_version=1`
- `Object` element => `enabled` defaults to `true`, `schema_version` defaults to `1`

### Schema version bump SOP

- Any structural change to a category normalization/payload contract must bump `schema_version` in backend manifest.
- This guarantees isolated cache invalidation per category.

### Cache key isolation (`cv`)

- Signature key material includes:
  - `...|cat=<categoryId>|cv=<categorySchemaVersion>|sig=<signatureBase>`
- Guarantees:
  - bumping `schema_version` for one category invalidates only that category cache
  - other categories remain unaffected

### Startup/offline safety default

- If manifest is not loaded yet (first launch/offline race), client uses:
  - `categorySchemaVersion = 1`
- This prevents accidental cache-wide invalidation from missing handshake data.

### Example

- `surprise_me` schema `1 -> 2` makes `surprise_me` cache MISS, while `crystal_waters` stays HIT.

## Cache Key Contract (Frozen Order)

Route cache key material uses a strict order and must remain stable:

`v=<cacheSchemaVersion>|norm=<normalizerVersion>|epoch=<cacheEpoch>|cat=<categoryId>|cv=<categorySchemaVersion>|sig=<signatureBase>`

Fields:
- `v`: cache schema version
- `norm`: trip normalizer version
- `epoch`: remote cache purge epoch from `/v1/manifest`
- `cat`: category id
- `cv`: per-category schema version from manifest
- `sig`: deterministic signature base from entry optimizer + normalized trip context

Changing order is a contract break and invalidates deterministic cache behavior.

## RFC-001 Frozen Protocol Guard

- `test/fixtures/rfc001_fixtures.json` is a frozen truth set.
- Any fixture change requires:
  - CODEOWNER approval,
  - explicit `protocol_version` or `norm_version` bump,
  - rationale update in docs (RFC-001 section).
- Cross-platform parity tests are merge-blocking:
  - `flutter test test/rfc001_parity_test.dart`
  - `flutter test test/rfc001_handshake_test.dart`
  - `node --test backend/test/rfc001_parity.test.js`
  - `node --test backend/test/rfc001_handshake.test.js`
- HandshakeManifest v1 is strict (no unknown fields): schema drift requires explicit version bump.
- Protocol changes require:
  - `protocol_version` or `norm_version` bump,
  - fixture update review (CODEOWNERS),
  - rationale update in docs.

## RFC-001 ErrorEnvelope v1

- All non-2xx backend errors are normalized to:
  - `ok: false`
  - `trace_id`
  - `error: { code, retryable, http_status, hint?, details? }`
- `x-trace-id` response header is always present and matches body `trace_id`.
- Rate-limit standard:
  - `code: E_RATE_LIMITED`
  - `details.retry_after_ms` is milliseconds (integer)
  - optional `Retry-After` header is seconds (integer).
- Flutter handles ghost HTML/empty upstream failures with synthetic `E_INTERNAL` (retryable) and never stores raw HTML body in details.
- Dev-only self-validation guard:
  - enabled when `RFC001_SELF_VALIDATE=1` or `NODE_ENV != production`
  - on validation failure: logs CRITICAL and returns safe fallback `E_INTERNAL` envelope (non-fatal).

## RFC-001 Capability Enforcement (VAL4 Bridge)

- Backend enforces category capability per operation (`read|write|delete`) and denies unknown/inactive categories.
- Client CV integrity is enforced for category-sensitive endpoints:
  - canonical header: `x-rn-cv`
  - category selector header: `x-rn-category`
- CV mismatch/missing returns `E_SCHEMA_MISMATCH` (`409`) with `cat`, `client_cv`, `server_cv`.

## VAL 6 Redis L2 Cache (Phase 1)

- Shared L2 cache is optional and serverless-safe:
  - enable with `REDIS_URL`
  - optional TLS via `REDIS_TLS=true`
  - global bypass switch: `CACHE_ENABLED=false`
- Cache-aside is integrated in `POST /v1/trip/generate`:
  - `OK` envelope caches successful payloads
  - `ERR` envelope caches transient upstream failures (negative caching)
- TTL policy:
  - `CACHE_OK_TTL_SECONDS` (default `300`)
  - `CACHE_ERR_TTL_SECONDS` (default `60`)
- Redis failures never crash requests:
  - graceful bypass to MISS
  - request continues without cache dependency.
- Singleflight contention control (Phase 2):
  - lock key uses Redis `SET NX PX` with tokenized release via Lua compare-delete
  - lock TTL defaults to `12s` (`CACHE_LOCK_TTL_MS`, baseline for serverless variability)
  - waiter budget defaults to `1200ms` (`CACHE_WAIT_BUDGET_MS`)
  - safety margin for remaining execution time defaults to `1000ms` (`CACHE_WAIT_SAFETY_MARGIN_MS`)
  - jittered poll schedule defaults to `40,80,150,250` ms (`CACHE_WAIT_POLL_SCHEDULE_MS`)
  - waiter timeout returns `E_DEGRADED_MODE` (`503`, retryable).
- Golden Store Sync (Phase 3, async L2 -> L3):
  - successful compute path can publish a best-effort Golden job (never blocks response)
  - global idempotency by `docId=sig` in Firestore (`create()` semantics)
  - shareability enforced via manifest capability (`shareable=false` => skip publish)
  - in-instance dedupe window (`GOLDEN_PUBLISH_DEDUPE_TTL_SECONDS`, default `30s`)
  - payload compression:
    - `json` for small payloads
    - `gzip+base64` above `GOLDEN_COMPRESS_THRESHOLD_BYTES` (default `2048`)
  - payload integrity:
    - `payload_hash = sha256(canonical JSON string)` verified in worker before write
  - retention:
    - `expires_at` stored per record (default `GOLDEN_TTL_SECONDS=2592000`, 30d)
    - records carry `epoch` in `key_meta` for epoch-aware cleanup policy.
  - config:
    - `GOLDEN_QUEUE_ENABLED` (default `true`)
    - `GOLDEN_FIRESTORE_ENABLED` (default `true`)
    - `GOLDEN_COLLECTION` (default `golden_routes`)
    - `GOLDEN_TTL_SECONDS` (default `2592000`)
    - `GOLDEN_COMPRESS_THRESHOLD_BYTES` (default `2048`)
    - `GOLDEN_PUBLISH_DEDUPE_TTL_SECONDS` (default `30`)
- Global L3 retrieval (Phase 4, leader-only):
  - on L2 miss, only singleflight leader may read Firestore by `docId=sig`
  - strict integrity guard required before serving:
    - protocol/norm/epoch/cv match
    - payload decode success
    - `sha256(canonicalJson) == payload_hash`
  - on valid hit, payload is backfilled to L2 best-effort
  - on L3 miss/error/invalid, flow falls back to normal compute (no request failure).
- Provider resilience (Phase 5):
  - provider calls use `AbortController` budget (`PROVIDER_BUDGET_MS`, default `650`)
  - effective budget should honor serverless remaining time: `min(PROVIDER_BUDGET_MS, remaining-150ms)`
  - circuit breaker with selective opening:
    - user-caused 4xx never opens breaker
    - provider/system failures (429/5xx/timeout) can open breaker
  - breaker tuning:
    - `CB_ENABLED`, `CB_STORAGE=memory|redis`
    - `CB_WINDOW_MS`, `CB_MIN_SAMPLES`, `CB_FAIL_RATIO_OPEN`
    - `CB_OPEN_MS`, `CB_HALF_OPEN_TRIALS`, `CB_HALF_OPEN_JITTER_MS`, `CB_KEY_PREFIX`.
  - optional chaos injection (non-production only):
    - `CHAOS_ENABLED=1`
    - `CHAOS_PROVIDER_FAIL_RATE=0.4`
    - `CHAOS_PROVIDER_MODE=rate_limit|timeout`
    - `CHAOS_RETRY_AFTER_MS=2000`
    - `CHAOS_PROVIDER_TIMEOUT_MS=5000`
    - ignored when `NODE_ENV=production`.

## VAL 6.2 Stress/Chaos Runner

Dual-lane load runner:

```bash
BACKEND_SHARED_TOKEN=... node tools/chaos_load_test.js http://localhost:8787
```

Full 3-terminal runbook:
- `docs/val6_2_fire_drill_runbook.md`
- Quick 2-minute dry-run:
  - `docs/val6_2_fire_drill_quick_run.md`

Defaults:
- total 10 min
- repeat lane 35 RPS
- unique lane 15 RPS
- phases: warmup/chaos/recovery (20%/50%/30%)

Optional tuning:
- `CHAOS_TEST_DURATION_MS`
- `CHAOS_WARMUP_MS`
- `CHAOS_PHASE_MS`
- `CHAOS_REPEAT_RPS`
- `CHAOS_UNIQUE_RPS`

Telemetry heartbeat correlator (separate terminal):

```bash
node backend/server.js 2>&1 | rg -N '"event":"(cb_state_change|cb_call_blocked|provider_call_failure|provider_call_success|cache_hit|cache_miss|l3_hit|l3_backfill_ok|waiter_timeout)"'
```
- Node micro-performance metric:
  - `node --test backend/test/rfc001_perf.test.js`
  - emits `RFC001_CANON_P50_US=<value>`
  - conservative default threshold: `5000us` (5ms), configurable via `RFC001_CANON_P50_THRESHOLD_US`
  - PR CI is non-blocking (warning only), main CI is blocking.
  - if CI noise appears: raise threshold (e.g. `20000us`) or move blocking enforcement to nightly.

## Diagnostics Redaction Policy

Diagnostics export is redacted and safe to share for debugging:
- No coordinates in diagnostics payload (`lat/lng/latitude/longitude`)
- No itinerary payload blocks (`itinerary`, destination names/addresses, stop names)
- No place identifiers (`placeId`, `place_id`)
- No user identifiers (`email`, `phone`, `deviceId`, advertising ids)

Release behavior:
- Diagnostics debug UI entry is shown only in debug builds
- `lastGoldenFailure` block is emitted only in debug/profile JSON export (`!kReleaseMode`)
- Release export omits golden-failure trace entirely

## Production Config Matrix

Source of truth: `backend/.env.example`.

| Variable | Default | Recommended Production | Notes |
| --- | --- | --- | --- |
| `ENRICHMENT_ENABLED` | `0` | `0` during rollout, then `1` | Enables VAL 7.3 enrichment stage. |
| `ALLOW_TEST_TOGGLES` | `0` | `0` | Must remain off in production. |
| `DISABLE_INTERNAL_RATE_LIMITS` | `0` | `0` | Dev/test-only and additionally gated in non-production code paths. |
| `PROVIDER_BUDGET_MS` | `650` | tune per SLO (for example `650-3000`) | End-to-end provider budget guard. |
| `JUDGE_ENABLED` | `0` | `0` during rollout, then `1` | Master flag for VAL 7.2 judge stage. |
| `JUDGE_ENABLED_FREE` | `0` | policy-dependent | Free-tier judge execution gate. |
| `JUDGE_ENABLED_PAID` | `1` | `1` | Paid-tier judge execution gate. |
| `JUDGE_STRICT_DEFAULT` | `0` | `0` unless explicitly required | Strict mode default switch. |
| `MAX_JUDGE_CALLS_PER_REQUEST` | `2` | `2` | Caps judge + re-judge calls. |
| `MAX_REPAIR_ATTEMPTS` | `1` | `1` | Caps auto-repair attempts. |
| `ENRICHMENT_CONCURRENCY` | `4` | `4` (or lower under pressure) | Parallel stop enrichment worker limit. |
| `ENRICHMENT_PER_CALL_TIMEOUT_MS` | `400` | `300-800` by provider SLA | Per enrichment provider request timeout. |

Notes:
- There is no `PROVIDER_TIMEOUT_MS` variable in current backend config; the effective guard is `PROVIDER_BUDGET_MS`.
- There is no dedicated `SOFT_STRICT_MODE` env variable; Soft Strict behavior is implemented in judge decision logic for strict mode.

## Telemetry Events

Event names below are extracted from current implementation.

Router (`backend/src/intelligence/router.js`, `[INTEL]` logs):
- `router_attempt`
- `router_success`
- `router_attempt_failed`
- `router_exhausted`

Pipeline (`backend/src/services/trip_generation_pipeline.js`, `emitTelemetry`):
- `INTELLIGENCE_ROUTER_FAILED`
- `INTELLIGENCE_ROUTER_SELECTED`
- `ARCHITECT_JSON_INVALID`
- `PLACE_FALLBACK_USED`
- `DEST_TOO_CLOSE_DROPPED`
- `STAY_NIGHTS_MISMATCH`
- `CALENDAR_SPAN_MISMATCH`
- `DETAILS_PROVIDER_FAILED`
- `PLAN_GENERATED`
- `judge_skipped`
- `judge_passed`
- `judge_failed`
- `repair_triggered`
- `repair_exhausted`
- `repair_success`
- `enrichment_skipped`
- `enrichment_success`
- `enrichment_partial`
- `enrichment_failed`

Caching / resilience logs:
- Cache (`backend/src/cache/cache_service.js`): `cache_bypass_redis_unavailable`, `cache_miss`, `cache_hit`, `cache_negative_hit`, `cache_write_ok`, `cache_write_err`
- Circuit breaker (`backend/src/providers/circuit_breaker.js`): `cb_state_change`
- Provider call wrapper (`backend/src/providers/provider_call.js`): `cb_call_blocked`, `provider_call_success`, `provider_call_failure`
- Golden/L3 (`backend/src/golden/*.js`): `l3_miss`, `l3_stale_invalidated`, `l3_hit`, `l3_read_fail`, `golden_write_failed`

## API Reference: POST /v1/trip/generate

Path:
- `POST /v1/trip/generate`

Headers:
- `x-roadnomai-token: <BACKEND_SHARED_TOKEN>`
- `x-rn-category: surprise_me`
- `x-rn-cv: 1`

Example request:

```json
{
  "requestId": "demo-req-001",
  "mode": "SURPRISE_ME",
  "category": "surprise_me",
  "seed": 42,
  "trip": {
    "totalDays": 5,
    "startDate": null
  },
  "policy": {
    "quality": "balanced"
  }
}
```

Example success response (`200`):

```json
{
  "schemaVersion": 1,
  "planId": "demo-req-001",
  "meta": {
    "mode": "SURPRISE_ME",
    "totalDaysRequested": 5,
    "startDate": null,
    "generatedAt": "2026-02-28T00:00:00.000Z",
    "seed": 42,
    "validation_mode": "lenient",
    "judge_passed": true,
    "repair_applied": false,
    "enrichment_skipped": true,
    "enrichment_reason": "disabled"
  },
  "destinations": [],
  "transportSegments": [],
  "timeline": [],
  "details": {
    "byDestinationId": {}
  },
  "warnings": []
}
```

Example error response (RFC-001 envelope):

```json
{
  "ok": false,
  "trace_id": "9f6f7f5f-5f36-49c6-aaf0-cc7d16c9f92f",
  "error": {
    "code": "E_RATE_LIMITED",
    "retryable": true,
    "http_status": 429,
    "hint": "Too many requests",
    "details": {
      "retry_after_ms": 60000
    }
  }
}
```

Determinism note:
- `taskId` is deterministically derived from sanitized semantic input + schema version (JCS + SHA-256).
- `taskId` is currently emitted in router telemetry (`[INTEL]`) and is not part of the public response body contract.

## Quick Start (Docker)

1. Prepare backend env file:

```bash
cp backend/.env.example backend/.env
```

2. Fill required secrets in `backend/.env` (`BACKEND_SHARED_TOKEN`, provider keys).

3. Start Redis + backend:

```bash
docker compose up --build
```

4. Health check:

```bash
curl -sS http://localhost:8787/healthz
```

## Deployment Notes

- Runtime requirements: Node.js 20+, Redis 7+.
- Backend is stateless and horizontally scalable; Redis is used for shared L2 cache/singleflight coordination.
- Budget discipline is enforced through provider resilience budget guards (`PROVIDER_BUDGET_MS`).
- Enrichment is designed as soft-fail: failures degrade only enrichment metadata, not core plan generation response.

## Getting Started

This project is a starting point for a Flutter application.

A few resources to get you started if this is your first Flutter project:

- [Lab: Write your first Flutter app](https://docs.flutter.dev/get-started/codelab)
- [Cookbook: Useful Flutter samples](https://docs.flutter.dev/cookbook)

For help getting started with Flutter development, view the
[online documentation](https://docs.flutter.dev/), which offers tutorials,
samples, guidance on mobile development, and a full API reference.
