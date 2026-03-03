# VAL 7.5 KPI Audit Artifacts

This folder stores compact audit outputs for VAL 7.5 synthetic KPI validation.

## Environment

- runtime: local synthetic mode (`/v1/trip/generate`)
- weather healthy mode: `mock`
- weather degraded mode: `real` + forced timeout + breaker short-circuit

## Profile

- `rps=50`
- `duration=5000ms` (smoke baseline)
- `warmup=1000ms`
- `synthetic_delay=200ms`

## Pass Gates

- healthy: `p95DeltaMs < 100`, `errorRate < 1%`
- degraded: post-warmup short-circuit within degraded limit, `errorRate < 1%`

## Files

- `val75_report_healthy.json`
- `val75_report_degraded.json`

`generatedAt` timestamps are informational and not gating fields.
