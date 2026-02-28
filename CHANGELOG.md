# Changelog

## [0.7.3-beta] - 2026-02-28

### Added
- VAL 7.1 Multi-Model Router with retryable-only fallback and provider routing telemetry.
- VAL 7.2 Judge + Repair flow with maximum one repair attempt per request.
- VAL 7.3 Media Enrichment as a non-breaking meta layer (`meta.enrichment`).
- Deterministic enrichment cache keys from canonicalized place input (`sha256`) with split TTL policy (geo 30d, images 7d).

### Changed
- Strict validation mode now uses Soft Strict semantics: warn-only outcomes do not hard-fail responses.
- Enrichment meta now exposes explicit soft-fail reasons (`disabled`, `budget`, `provider_disabled`, `timeout`).
- Release baseline documentation added for VAL 7.3 freeze and test artifacts.

### Fixed
- Deterministic task identity hardened: unstable fields (`requestId`, `trace*`, `timestamp`, `nonce`, generated fields) excluded from semantic fingerprints.
- Cache/singleflight determinism preserved across router + judge + enrichment paths.
- Image enrichment enforces attribution fields; unattributed images are rejected.

### Tested
- `node --test backend/test/judge_flow.test.js backend/test/intelligence_router.test.js backend/test/intelligence_integration.test.js backend/test/enrichment_flow.test.js`
- Result: `21/21 PASS` (archived at `artifacts/val7_3/test_output.txt`).

## Korak 4 – Nearby Explore & Launch Hygiene

- Added `nearby_explore_v1` and `review_mock_mode` feature flags (default OFF).
- Added strict Search A / Search B controller separation with scope guardrails.
- Added Nearby Explore local-first SearchBar B (debounced), category filtering, and category config with localization + icon fallback.
- Added review mock dataset mode for Nearby Explore (no network/GPS dependency).
- Added Korak 4 release docs and release checklist template.
