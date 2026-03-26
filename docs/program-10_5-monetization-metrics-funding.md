# Program 3 — Phase 10.5 / Monetization, Metrics, and Funding Package

## Goal
Define the narrowest truthful monetization, metrics, retention, and funding package that the current repo can support without implying live affiliate attribution, verified commercial truth, or validated market traction.

## Current commercial reality
- The canonical package is still Android-first shared-shell partner onboarding.
- The current hotel commercial path is Booking.com direct web outbound only.
- Booking affiliate permission is not yet available.
- Premium status remains abstraction-first and provider-aware, not provider-verified commercial truth.
- GTM structure exists, but repo-backed real market signal is still thin.

## Monetization minimum

### Commercially real now
- A user can tap a hotel CTA from trip context and RoadNomai can build a deterministic, non-mutating Booking.com outbound URL.
- Canonical journey context is preserved around the outbound flow through `lineage_id`, `variant_id`, optional `stop_id`, `category_key`, and source-surface attribution.
- Funnel and telemetry surfaces can measure outbound intent and outbound open/fail behavior.

### Structured but not yet commercially proven
- Tenant-specific hotel outbound routing through `booking_com`.
- Premium activation posture and billing-provider context.
- Commercial-intent instrumentation and funnel aggregation.

### Blocked until affiliate permission and verified commercial truth exist
- Affiliate parameter activation.
- Booking confirmation truth.
- Conversion attribution.
- Commission, settlement, or payout reporting.
- Reconciliation and support-safe commercial correction flows.

### Out of scope for the current package
- Dedicated branded binary monetization paths.
- Custom providers.
- Verified auth or billing/store activation claims.
- Control-plane or reconciliation-backed partner reporting.

## Booking outbound mode
- Current mode: `direct_web`
- Current evidence class: `CODE-BACKED` + `TEST-BACKED`
- Current commercial classification: `DIRECT WEB ONLY` + `NO VERIFIED COMMERCIAL TRUTH`

Important interpretation rule:
- The existing `activation_state = affiliate_ready` contract label means the outbound contract preserves enough canonical context for a later affiliate activation layer.
- It does not mean affiliate parameters, attribution, booking confirmation, payout, or reconciliation are live.

Future activation rule:
- A future `affiliate` mode should be treated as blocked until Booking affiliate permission exists and verified callback/reconciliation truth exists.
- Until then, the safe fallback is the current direct Booking.com web path with no affiliate-only behavior assumed.

## Minimal usage metrics surface

### Query surface
- `GET /v1/analytics/funnel-summary`
- Prometheus metric families for funnel, outbound, commercial intent, and leakage

Interpretation rule:
- This surface is a real measurement capability, but the repo does not yet contain a real pilot cohort or durable historical dataset that would turn those metrics into traction proof by itself.

### Measured today
| Metric | Source | Evidence class | Interpretation boundary |
| --- | --- | --- | --- |
| `saved_to_generated_ratio` | `backend/src/services/funnel_analytics.js` + funnel summary route | `CODE-BACKED` + `TEST-BACKED` | local save intent only, not sync confirmation |
| `execution_start_rate` | `backend/src/services/funnel_analytics.js` + funnel summary route | `CODE-BACKED` + `TEST-BACKED` | execution intent only, not journey completion |
| `affiliate_click_rate` | `backend/src/services/funnel_analytics.js` + funnel summary route | `CODE-BACKED` + `TEST-BACKED` | outbound click intent only, not partner conversion attribution |
| `return_to_saved_trip_rate` | `backend/src/services/funnel_analytics.js` + funnel summary route | `CODE-BACKED` + `TEST-BACKED` | local saved-trip reopen/open proxy only |
| `roadnomai_booking_outbound_total` | Prometheus telemetry | `CODE-BACKED` + `TEST-BACKED` | outbound tap/open/fail counts, not booking completion |
| `roadnomai_commercial_intent_total` | Prometheus telemetry | `CODE-BACKED` + `TEST-BACKED` | intent signal only |

### Missing today
- persistent historical analytics warehouse
- retained cohort history
- verified partner conversion reporting
- attributed revenue reporting
- provider-backed payout or reconciliation exports

## Retention minimum
- Current retention minimum is a saved-trip return proxy, not a cohort-retention claim.
- The measurable retention proxy is `return_to_saved_trip_rate`.
- That metric only counts explicit reopen/open behavior when the destination variant is locally marked as saved.
- The repo does not currently evidence a real retained partner cohort, a historical retention curve, or a commercially meaningful repeat-booking loop.

Retention interpretation:
- Current classification: `STRUCTURED BUT UNPROVEN`
- Evidence class: `CODE-BACKED` + `TEST-BACKED`
- Not yet supported: `RETENTION-MEASURED` in the investor or operator sense

## Canonical funding narrative

### What the product is
RoadNomai is an Android-first shared-shell travel planning package with tenant-aware branding, locally conservative restore semantics, partner-ready hotel outbound, and a measured execution/funnel surface.

### What has been de-risked
- local save/reopen and fallback restore discipline
- Android-first partner onboarding package
- tenant-aware package boundaries
- direct Booking outbound intent path
- minimal pilot first-response ops posture
- bounded funnel and outbound telemetry surface

### What remains open
- verified tenant auth
- provider-backed premium truth
- callback verification and reconciliation
- Booking affiliate permission and affiliate activation
- strong GTM traction proof
- durable retention evidence

### What is intentionally not claimed
- revenue
- booking confirmation truth
- conversion attribution
- payout or commission reporting
- verified billing activation
- validated retention curve
- broad cross-platform parity

## Claims-to-evidence mapping
| Claim | Current posture | Evidence class | Funding classification |
| --- | --- | --- | --- |
| Booking outbound works as a non-mutating hotel referral path | implemented | `CODE-BACKED` + `TEST-BACKED` | `STRUCTURED BUT UNPROVEN` |
| Booking outbound is affiliate monetization | not supported | no valid evidence | `KNOWN LIMITATION` |
| Premium posture exists | implemented as abstraction | `CODE-BACKED` + `TEST-BACKED` | `STRUCTURED BUT UNPROVEN` |
| Premium posture is provider-backed commercial truth | not supported | no valid evidence | `KNOWN LIMITATION` |
| Funnel summary and outbound-intent metrics exist | implemented | `CODE-BACKED` + `TEST-BACKED` | `USAGE-MEASURED` |
| Retention is measured as a real cohort outcome | not supported | no valid evidence | `STRUCTURED BUT UNPROVEN` |
| GTM traction is proven | not supported | no valid evidence | `COMMERCIAL SIGNAL THIN` |
| Funding narrative is bounded by repo truth | supported by current package/evidence docs | `DOC-BACKED` + `CODE-BACKED` + `TEST-BACKED` | `FUNDING-NARRATIVE SUPPORTED` |

## Next milestone
The next funding-relevant milestone is to move the hotel commercial path from direct-web intent into verified commercial truth by securing Booking affiliate permission and then implementing an optional affiliate mode with verified callback/reconciliation evidence, while keeping the current direct-web fallback intact until that work is complete.

## Carry-forward limitations that must stay explicit
- Android-first shared-shell package remains the canonical package.
- iOS preferred external `/saved/[id] -> /trip` remains constrained.
- Fresh non-dev cross-platform provenance is still not fully runtime-backed.
- GTM remains structured but not yet strongly evidenced by real partner response.
- Current pilot ops/support is minimal and explicit, not scaled operations.
- Dedicated branded binary, verified auth/commercial truth, custom providers, and control-plane work remain outside the canonical package.
