# Contributing

## Development Principles

- Preserve protocol and schema contracts (RFC-001 and category/version invariants).
- Prefer additive changes and backward-compatible migrations.
- Keep cache/task determinism intact (no volatile fields in semantic fingerprints).
- Follow VAL freeze discipline: do not regress frozen layers without explicit rationale.

## Schema-First Rules

- Update schemas before (or together with) behavior changes.
- If response shape changes, update tests and docs in the same PR.
- For protocol-sensitive changes, include versioning rationale.

## Required Validation Before PR

From repo root:

```bash
flutter analyze
flutter test test/rfc001_parity_test.dart
flutter test test/rfc001_handshake_test.dart
node --test backend/test/rfc001_parity.test.js
node --test backend/test/rfc001_handshake.test.js
```

For VAL 7 freeze baseline:

```bash
node --test \
  backend/test/judge_flow.test.js \
  backend/test/intelligence_router.test.js \
  backend/test/intelligence_integration.test.js \
  backend/test/enrichment_flow.test.js
```

## Commit / Release Hygiene

- Keep diffs focused and minimal.
- Do not commit secrets, local absolute paths, or machine-specific config.
- Update `CHANGELOG.md` for release-relevant changes.
- Keep release checklist in `.github/RELEASE_TEMPLATE.md` aligned with current VAL freeze status.
