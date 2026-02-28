# Korak 3 — Production Hardening

Korak 3 je omejen na **stabilnost, resilienco in observability**. Brez novih produktnih feature-jev.

## Freeze Policy

`Allowed`:
- test hardening (`contract`, `golden`, `boundary`, `smoke`)
- observability (strukturirani log dogodki brez PII)
- robustnost (`timeout`, `retry`, `guardrails`)
- docs + evidence template

`Disallowed`:
- response schema spremembe
- nova poslovna logika za generiranje destinacij/timeline/order
- UX/product spremembe
- dodatni API side-effecti

## Korak 3 DoD

- [ ] `backend/src/services/itineraryEnrichCore.js` ostane single source of truth.
- [ ] Premium cap/fallback deluje v produkcijskem `/v1/itinerary/enrich`.
- [ ] `npm test` green.
- [ ] `npm run test:contract` green.
- [ ] `npm run test:golden` green, ko je `GOOGLE_MAPS_API_KEY` prisoten.
- [ ] `npm run smoke:enrich` uspe (4 scenariji x free/premium) z global timeout 30s.
- [ ] Warnings ostanejo na known allowlist-u.
- [ ] Shape hash invariant test green.
- [ ] Structured logs vključeni brez PII.

## Local Commands

```bash
cd backend
npm test
npm run test:contract
npm run test:golden
npm run smoke:enrich
```

`smoke:enrich` bere ključ iz `GOOGLE_MAPS_API_KEY` ali `GOOGLE_MAPS_SERVER_API_KEY` (env/.env).
