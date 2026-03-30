# RoadNomai

RoadNomai is a modular travel planning platform for AI-assisted itinerary generation and partner-facing travel experiences. It is designed to support **white-label**, **B2B**, and **B2C** product models from a shared core.

## Public-safe description

Publicly, RoadNomai should be described narrowly as:

- a shared platform core for travel planning and trip handling
- a partner-aware configuration model rather than separate product codebases
- a system intended to support white-label, B2B, and B2C directions from the same foundation
- an engineering posture that emphasizes bounded scope, verification discipline, and explicit status claims

This wording is the safe GitHub/public description baseline.
It does not imply commercial proof, full automation, full cross-platform parity, or protected-boundary closure.

## Repository scope

This repository is the private working repository.
It contains both public-safe platform material and internal governance/status documents.

Public-facing description should stay limited to the public-safe description above.
Internal authority, governance, and evidence-tracking documents in this repository are not public marketing copy and should not be treated as public status authority.

## What RoadNomai supports

The platform is intended to support:

- **white-label deployments** for agencies and partners
- **B2B partner-aware product models**
- **B2C branded product experiences**
- modular client and backend surfaces built on a common core
- configurable branding, routing, and platform posture
- observability and verification-oriented engineering workflows

Core capabilities include:

- AI-assisted itinerary generation
- multi-country and multi-destination trip planning
- destination replacement without breaking route structure
- route flow and transport-context support
- nearby and related discovery workflows
- white-label, B2B, and B2C platform support

## Documentation map

### Public-safe platform docs

- [`docs/platform-overview.md`](docs/platform-overview.md)
- [`docs/architecture-overview.md`](docs/architecture-overview.md)
- [`docs/white-label-overview.md`](docs/white-label-overview.md)
- [`docs/b2b-overview.md`](docs/b2b-overview.md)
- [`docs/b2c-overview.md`](docs/b2c-overview.md)
- [`docs/feature-matrix.md`](docs/feature-matrix.md)
- [`docs/public-status.md`](docs/public-status.md)
- [`docs/deployment-boundaries.md`](docs/deployment-boundaries.md)

### Public technical references

- [`docs/DEVELOPER_GUIDE.md`](docs/DEVELOPER_GUIDE.md)
- [`backend/docs/observability.md`](backend/docs/observability.md)

### Release-oriented public notes

- [`docs/VAL_8_0_PUBLIC_RELEASE.md`](docs/VAL_8_0_PUBLIC_RELEASE.md)

### Current status and program snapshots

- [`docs/program-10_5-master-status.md`](docs/program-10_5-master-status.md)
- [`docs/phase9plus-execution-summary.md`](docs/phase9plus-execution-summary.md)

### Program 10.5 partner-layer docs

- [`docs/program-10_5-partner-onboarding-package.md`](docs/program-10_5-partner-onboarding-package.md)
- [`docs/partner-health-model.md`](docs/partner-health-model.md)

### Internal authority and governance docs

- [CURRENT_TRUTH.md](CURRENT_TRUTH.md)
  Canonical internal current-status authority.
- [partner_preflight_checklist.md](partner_preflight_checklist.md)
  Operational pre-activation control for partner go/no-go review.
- [phase13-14-boundary.md](phase13-14-boundary.md)
  Internal boundary/governance reference for later-phase claim discipline.
- [real_signal_evidence_tracker.md](real_signal_evidence_tracker.md)
  Internal tracker for real outreach, reply, pilot, and activation evidence only.
- [AGENTS.md](AGENTS.md)
  Repository-level Codex guardrails.

## Public repo alignment

If material from this repository is mirrored into a public repository:

- keep only the public-safe platform description and public-safe docs
- avoid internal status/governance docs as public authority
- avoid partner-specific, operational, control-plane, or sensitive commercial material
- do not turn internal readiness structure into public traction claims

## Working rule

For internal work:

- use [CURRENT_TRUTH.md](CURRENT_TRUTH.md) first for status
- use [partner_preflight_checklist.md](partner_preflight_checklist.md) for partner activation review
- use [phase13-14-boundary.md](phase13-14-boundary.md) for claim-boundary discipline
- use [real_signal_evidence_tracker.md](real_signal_evidence_tracker.md) only when real external signal actually exists
