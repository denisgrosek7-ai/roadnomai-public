# Security Policy

## Supported Scope

This policy covers code and configuration in this repository.

## Reporting a Vulnerability

- Please do not open public GitHub issues for security vulnerabilities.
- Report privately to the repository maintainers with:
  - affected component/path
  - reproduction steps
  - impact assessment
  - suggested mitigation (if available)

Maintainers will acknowledge receipt and coordinate remediation and disclosure timing.

## Response SLA

- Initial acknowledgment target: within 72 hours.
- Triage status update target: within 7 calendar days.

## Security Posture (Current)

- Provider/API keys are expected in environment variables and must never be committed.
- Backend auth is enforced via shared token header (`x-roadnomai-token`).
- RFC-001 error envelopes avoid leaking raw upstream payloads.
- Cache key fingerprints are deterministic and exclude volatile transport metadata.
- Enrichment is soft-fail and does not block core plan generation.
- Diagnostics export redacts sensitive payload classes (coordinates, place ids, user identifiers).

## Hard Requirements

- Do not store provider secrets in cache payloads.
- Do not log raw secret values.
- Keep `ALLOW_TEST_TOGGLES=0` in production.
- Keep `DISABLE_INTERNAL_RATE_LIMITS=0` in production.
