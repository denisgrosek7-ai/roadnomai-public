# Program 3 — Phase 10.5 / Partner Health Model

This document defines the minimum partner-specific health and alerts model that the current repository can support without inventing a control plane.

It is a classification model and setup discipline.
It is not proof of live per-partner alert routing, automated health dashboards, or automated provisioning.

## Purpose

Use this model once more than one tenant or partner is present so operators can distinguish:

- global shared-core failures
- partner-specific config or routing failures
- known package limitations
- support issues that are still inside the canonical Android-first package

Use it together with:

- [program-10_5-partner-onboarding-package.md](/Users/denisgrosek/RoadNomai/docs/program-10_5-partner-onboarding-package.md)
- [program-10_5-pilot-ops-support.md](/Users/denisgrosek/RoadNomai/docs/program-10_5-pilot-ops-support.md)

## Health record model

| Field | Source | Evidence basis | Reality | Interpretation |
| --- | --- | --- | --- | --- |
| `partner_id` | `partner_manifest.partner_id` | DOC-BACKED + CONFIG-BACKED | MANUAL | Canonical partner-layer identifier |
| `tenant_key` | `partner_manifest.tenant_key` | DOC-BACKED + CONFIG-BACKED | MANUAL | Setup identifier that must map to `tenant_config.tenant_id` |
| `config_loaded` | `/v1/tenant/context` and `TENANT_REGISTRY` | CODE-BACKED + CONFIG-BACKED | MANUAL CHECK | `true` only after the tenant resolves without unknown-tenant fallback |
| `runtime_healthy` | Android shared-shell runtime checks and support evidence | DOC-BACKED + RUNTIME-BACKED | MANUAL CHECK | Use only for the canonical Android path; do not infer iOS parity |
| `backend_healthy` | `/readyz`, `/metrics`, Prometheus targets | CODE-BACKED + CONFIG-BACKED | QUERYABLE, MANUAL INTERPRETATION | Shared backend health for the active partner flow |
| `provider_degraded` | provider circuit breaker keyed by tenant and provider | CODE-BACKED | QUERYABLE IN CODE, MANUAL OPERATIONS VIEW | Indicates `partner_unavailable` posture for the tenant/provider pair |
| `premium_mismatch` | premium entitlement payload and degraded posture | CODE-BACKED | MANUAL CHECK | Signals degraded or stubbed premium posture, not verified billing truth |
| `outbound_failing` | partner-ready failure path, outbound metrics, incident evidence | CODE-BACKED + DOC-BACKED | MANUAL CHECK | Use for direct-web Booking failures or stub-provider failures |
| `last_activity_at` | funnel and outbound telemetry timestamps | CODE-BACKED | PARTIALLY SUPPORTED | No dedicated per-partner dashboard; derive only when evidence exists |
| `last_critical_error` | incident record or support evidence | DOC-BACKED | MANUAL | Must come from support capture, not inferred from silence |
| `alert_status` | operator classification | DOC-BACKED | MANUAL | `healthy`, `triage_required`, `known_limitation`, `out_of_scope`, or `paused` |

## Minimum health sources

Use these sources first:

- `GET /readyz`
- `GET /metrics`
- `GET /api/v1/targets` on Prometheus
- `GET /v1/tenant/context`
- `GET /v1/analytics/funnel-summary`
- provider circuit state from `backend/src/services/hardening_service.js`
- incident/support evidence captured through [program-10_5-pilot-incident-template.csv](/Users/denisgrosek/RoadNomai/docs/program-10_5-pilot-incident-template.csv)

## Global versus partner-specific interpretation

### Global shared-core issue

Treat the issue as global when any of the following is true:

- `/readyz` fails for all tenants
- Prometheus targets show shared backend failure
- persistence, migration, or trip-generation alerts are not tenant-specific
- Android shared-shell runtime is unavailable independent of tenant selection

### Partner-specific issue

Treat the issue as partner-specific when any of the following is true:

- `/v1/tenant/context` fails only for one tenant key
- category or routing policy rejects a request only for one tenant
- provider circuit posture opens only for one `tenant_id` plus provider pair
- outbound behavior fails only for one active partner setup
- a premium posture mismatch appears only for one tenant context

## Partner issue classes

Use these classes in support records and reviews:

- `config_setup_failure`
- `runtime_navigation_issue`
- `restore_open_issue`
- `provider_outbound_failure`
- `degraded_mode`
- `premium_mismatch`
- `known_platform_limitation`
- `out_of_scope_request`

## Manual versus already surfaced

### Already surfaced in code or monitoring

- backend readiness state
- backend metric families
- Prometheus target health
- provider circuit breaker status keyed by tenant and provider
- tenant context resolution failures

### Manual but explicit

- mapping a partner incident to `partner_id`
- setting `alert_status`
- recording `last_critical_error`
- deciding whether an issue is partner-specific or global
- deciding whether a known limitation has exceeded the disclosed support boundary

## Known limitation handling

- The iOS preferred external `/saved/[id] -> /trip` caveat stays a `known_platform_limitation` while it behaves inside the already disclosed boundary.
- Fresh non-dev provenance not being fully runtime-backed remains a package-wide known limitation, not a per-partner surprise incident.
- Booking stays `direct_web` only until affiliate permission and later implementation exist; missing affiliate attribution is not a partner-specific outage.

## What this model does not prove

- live per-partner alert routing
- automated partner dashboards
- automated partner activation or deactivation
- provider-backed commercial truth
- partner-isolated runtime infrastructure
