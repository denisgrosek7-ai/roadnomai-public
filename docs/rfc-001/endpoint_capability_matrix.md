# RFC-001 Endpoint Capability Matrix

Snapshot source: `backend/server.js` (current `main` workspace state).

## Guard Legend
- `CapabilityGuard`: `enforceCapability({ cat, operation }, manifest)`
- `CvGuard`: `enforceCv({ cat, clientCv }, manifest)`
- Category headers:
  - `x-rn-category` (canonical category selector)
  - `x-rn-cv` (canonical client category schema version)

## Matrix
| Endpoint | Method | Category source | Operation | Requires `x-rn-cv` | Guarded | Denial/Mismatch error codes |
|---|---|---|---|---|---|---|
| `/v1/manifest` | `GET` | N/A | N/A | No | No | N/A |
| `/v1/maps/place/textsearch` | `GET` | N/A | N/A | No | No | N/A |
| `/v1/maps/place/nearbysearch` | `GET` | N/A | N/A | No | No | N/A |
| `/v1/maps/place/details` | `GET` | N/A | N/A | No | No | N/A |
| `/v1/maps/geocode` | `GET` | N/A | N/A | No | No | N/A |
| `/v1/maps/directions` | `GET` | N/A | N/A | No | No | N/A |
| `/v1/ai/gemini` | `POST` | N/A | N/A | No | No | N/A |
| `/v1/ai/openai/responses` | `POST` | N/A | N/A | No | No | N/A |
| `/v1/trip/generate` | `POST` | `x-rn-category` header, fallback `body.category/body.categoryId/body.mode` | `write` | Yes (`x-rn-cv` preferred, `body.client_cv` fallback) | Yes (`CapabilityGuard` + `CvGuard`) | `E_CAPABILITY_DENIED` (403), `E_SCHEMA_MISMATCH` (409) |
| `/v1/transfer-info` | `GET` | N/A | N/A | No | No | N/A |
| `/v1/itinerary/enrich` | `POST` | `x-rn-category` header, fallback `body.category/body.categoryId/body.mode` | `write` | Yes (`x-rn-cv` preferred, `body.client_cv` fallback) | Yes (`CapabilityGuard` + `CvGuard`) | `E_CAPABILITY_DENIED` (403), `E_SCHEMA_MISMATCH` (409) |

## Deterministic Guard Behavior (current)
- Unknown category: denied with `E_CAPABILITY_DENIED` (403).
- Category not active in manifest: denied with `E_CAPABILITY_DENIED` (403).
- Mode `disabled`: denied for all operations.
- Mode `read_only`: `read` allowed, `write/delete` denied.
- Missing/invalid `client_cv`: `E_SCHEMA_MISMATCH` (409).
- `client_cv != server_cv`: `E_SCHEMA_MISMATCH` (409).

## ErrorEnvelope Contract for Guard Denials
Guard denials are emitted via central `sendErrorEnvelope(...)` and include:
- `ok: false`
- `trace_id` + `x-trace-id` header parity
- `error.code` in RFC-001 enum
- `error.details` with guard context:
  - capability: `cat`, `operation`, `mode`, `visibility`
  - cv mismatch: `cat`, `client_cv`, `server_cv`

## Update SOP (for new category-sensitive endpoints)
1. Add endpoint row to this matrix.
2. Assign operation (`read|write|delete`).
3. Wire `CapabilityGuard` + `CvGuard` before business logic.
4. Ensure client sends `x-rn-category` + `x-rn-cv` (or documented fallback).
5. Add/extend RFC001 guard tests.
