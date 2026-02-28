# Changelog

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
