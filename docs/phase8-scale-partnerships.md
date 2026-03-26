# Phase 8 Scale Partnerships

## Phase 8 posture
- Booking.com from Phase 7 remains the primary direct-web outbound for hotel/stay, with affiliate contract readiness preserved but not activated.
- Phase 8 extends the outbound contract model to additional verticals without activating a marketplace.

## Supported partner-ready verticals
- `stay_hotel`
- `car_rental`
- `travel_insurance`
- `local_experiences`

## Providers
- Active direct-web outbound:
  `booking_com` for `stay_hotel`
- Affiliate contract readiness only:
  `booking_com` preserves canonical outbound context, but attribution, settlement, and booking-confirmed truth are still not live
- Stub partner-ready:
  `car_rental_stub`, `insurance_stub`, `experiences_stub`

## Canonical context retained
- `trip_id`
- `lineage_id`
- `variant_id`
- optional `stop_id`
- `category_key`
- sync/restore caveat metadata when available

## Failure posture
- Unsupported vertical/provider combinations fail deterministically.
- Insufficient destination/place context fails deterministically.
- Partner-ready outbound never becomes a mutation path.

## Controlled exception
- Non-hotel verticals are stub outbound builders only in Phase 8.
