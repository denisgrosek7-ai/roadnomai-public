# Phase 7 Affiliate-Ready Outbound Contracts

Phase 7 does not activate affiliate attribution or settlement.

It introduces a partner-ready outbound bridge for Booking.com that preserves canonical journey context and leaves room for later affiliate parameter activation without redesigning contracts.

Canonical implementation:

- mobile schema: `apps/mobile-expo/src/api/affiliate.schemas.ts`
- mobile builder: `apps/mobile-expo/src/utils/bookingOutbound.ts`
- backend validator / builder: `backend/src/services/affiliate_service.js`
- backend route: `POST /v1/outbound/booking`

## AffiliateIntent

Required fields:

- `partner = booking_com`
- `target`
- `source_surface`
- `place_context`
- `attribution.slot`

Target keeps canonical identity:

- `trip_id` when available
- `lineage_id`
- `variant_id`
- `itinerary_version` when available
- `stop_id` when available

Optional context:

- `category_key`
- `attribution.journey_ref`
- `place_context.destination_label`
- `place_context.place_id`
- `place_context.lat/lng`
- `place_context.check_in_iso/check_out_iso`
- restore caveat metadata (`sync_state`, `stale_reason`, `snapshot_origin`)

Validation rules:

- unknown `partner` fails
- unknown `source_surface` fails
- unknown `attribution.slot` fails
- invalid target identity fails through canonical execution-target validation
- insufficient destination context fails
- invalid or partial date window fails

## AffiliateReadyOutbound payload

Returned fields:

- `partner = booking_com`
- `activation_state = affiliate_ready`
- `target`
- `source_surface`
- `category_key`
- `stop_id`
- `destination_label`
- `query_label`
- `outbound_url`
- `attribution_tag`
- `journey_ref`
- `check_in_iso`
- `check_out_iso`
- `location_anchor_kind`
- minimal echoed `place_context`

Contract decision:

- `outbound_url` is provider-safe and direct.
- canonical journey identity is preserved in the contract and events, not encoded as planner-mutating state.
- `activation_state = affiliate_ready` is a contract-readiness label only; it does not mean affiliate attribution, booking confirmation, payout, or reconciliation are live.

Controlled exception:

- Google `place_id` is preserved in contract metadata but not used as Booking.com provider truth.
