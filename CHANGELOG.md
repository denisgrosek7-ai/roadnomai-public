# Changelog

## VAL 8.0 — Production Hardening and Observability

- Added shared generation memory with global inventory reuse and per-user/per-device anti-repeat history.
- Added Postgres-backed persistence for generation inventory and history, with file-store fallback retained for local development paths.
- Added Redis-backed cache and coordination helpers with safe degraded fallback behavior.
- Added readiness semantics that distinguish Postgres hard failure from Redis degraded mode.
- Added Prometheus metrics, alert rules, Grafana provisioning, and dashboard definitions for business savings, infra health, and app reliability.
- Added multi-instance verification tooling and observability docs for local operational drills.

## VAL 7.5 — Context Stability Validation

- Added deterministic KPI isolation for weather context overhead measurement.
- Healthy KPI gate: mock/cache-only weather path, PASS when `p95DeltaMs < 100` and `errorRate < 1%`.
- Degraded KPI gate: forced timeout + breaker short-circuit, PASS on post-warmup budget.
- Added audit artifacts for healthy and degraded KPI reports.

## Korak 4 – Nearby Explore & Launch Hygiene

- Added `nearby_explore_v1` and `review_mock_mode` feature flags (default OFF).
- Added strict Search A / Search B controller separation with scope guardrails.
- Added Nearby Explore local-first SearchBar B (debounced), category filtering, and category config with localization + icon fallback.
- Added review mock dataset mode for Nearby Explore (no network/GPS dependency).
- Added Korak 4 release docs and release checklist template.

## VAL 7.4 — Dynamic Weather Context

- Integrated Google Maps Platform Weather API (backend-only).
- Context-aware grid caching (0.1° precision, deterministic keys).
- Semantic ContextPack v7.4 (LLM-friendly weather abstraction).
- Isolated circuit breaker (`cb:google:weather`).
- Hard timeout ≤ 500ms.
- Strict soft-fail policy (never blocks generation).
- Full test coverage: cache determinism, breaker isolation, soft-fail integrity.
