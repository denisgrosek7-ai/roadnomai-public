# RoadNomai VAL 8.0: Public Release Notes

This document is written for public GitHub usage. It is safe to reuse as:

- a GitHub Release body
- a PR summary
- a README excerpt

It intentionally avoids secrets, private infrastructure details, and unsupported performance claims.

## Short release summary

VAL 8.0 hardens the RoadNomai backend for shared-state, multi-instance operation and adds first-class observability for generation, reuse, readiness, and degraded-mode behavior.

Core themes in this release:

- shared generation memory
- Postgres-backed persistence
- Redis-assisted cache and coordination
- Prometheus, Grafana, and alerting support
- safer readiness and degraded-mode semantics

## Full public release notes

### What shipped

#### 1. Shared generation memory

RoadNomai now persists reusable generation results in a shared memory layer instead of relying only on local file-based state.

This includes:

- global inventory reuse for compatible requests
- per-user and per-device anti-repeat history
- explicit reuse modes for:
  - full plan reuse
  - destination-pool recomposition
  - fresh generation

This reduces avoidable regeneration and makes behavior more consistent across backend instances.

#### 2. Postgres plus Redis architecture

The backend now supports a production-oriented persistence model:

- Postgres as the durable source of truth
- Redis as an optional cache and coordination layer
- file adapter retained for development and fallback scenarios

The runtime also supports degraded behavior:

- if Redis is unavailable, correctness is preserved through Postgres-backed paths
- if Postgres is unavailable, readiness fails rather than pretending the service is healthy

#### 3. Observability as code

The repository now includes a full monitoring bootstrap layer:

- `/metrics` Prometheus endpoint
- Prometheus scrape config
- alert rules
- Grafana datasource provisioning
- Grafana dashboards for:
  - overview
  - business savings
  - infra health
  - app reliability

Metrics cover areas such as:

- generation lookups and reuse
- readiness state transitions
- Redis degraded periods
- Postgres health and latency
- LLM request and fallback behavior
- replace flow outcomes

### Why it matters

- backend instances can now share reusable inventory safely
- anti-repeat behavior is no longer tied to a single process
- degraded-mode behavior is explicit and observable
- local and production-style diagnostics are much easier

## Suggested GitHub release body

```md
## RoadNomai VAL 8.0: Production Hardening and Observability

VAL 8.0 upgrades the backend from a local file-oriented prototype model to a shared-state architecture with stronger persistence, safer degraded-mode behavior, and first-class observability.

### Highlights

- Shared generation memory with global inventory reuse and anti-repeat history
- Postgres-backed generation persistence with Redis-assisted cache and coordination
- `/readyz` semantics that distinguish Postgres failure from Redis degraded mode
- Prometheus metrics, Grafana dashboards, and alert rules included in the repo
- Multi-instance verification tooling for local operational drills

### Included areas

- generation memory and reuse orchestration
- replace-flow hardening
- readiness and degraded-mode handling
- observability bootstrap and dashboards
- backend verification tooling
```

## Suggested PR description

```md
## Summary

This PR introduces the VAL 8.0 backend hardening and observability layer.

## Included

- shared generation memory with Postgres and file adapters
- Redis-backed cache and coordination helpers
- readiness semantics for Postgres-required / Redis-optional operation
- optimistic locking for canonical Replace mutations
- Prometheus metrics, Grafana provisioning, and alert rules
- multi-instance verification tooling and focused backend tests
- replace race verification against the real refresh contract

## Operational impact

- enables safer multi-instance backend operation
- reduces avoidable regeneration through shared reuse paths
- improves observability for degraded mode, readiness, replace conflicts, and reuse behavior
```

## Suggested Public PR Comment

```md
## Summary

This PR hardens the RoadNomai backend for shared-state, multi-instance operation and improves operational visibility around generation, reuse, readiness, and Replace mutations.

## Included

- shared generation memory with Postgres-backed persistence
- Redis-assisted cache and coordination with degraded-mode safety
- optimistic locking for canonical Replace mutations
- readiness semantics for Postgres-required / Redis-optional operation
- Prometheus metrics, Grafana provisioning, and verification tooling

## Validation

- sustained-load validation completed successfully
- Redis degraded validation completed without backend crashes
- Replace optimistic-locking verification now covers stale-version and race scenarios through the real refresh contract
```

## Suggested Short PR Comment

```md
## Validation Summary

- Sustained-load validation completed successfully.
- Reuse and recomposition handled most successful generate requests.
- Modeled savings increased during the run.
- Redis was taken down and restored during live traffic without restarting backend instances.
- Readiness correctly switched to degraded mode during outage and recovered afterward.
- Generate and reuse traffic continued through Postgres-backed paths.
- Replace optimistic-locking verification covers stale-version and concurrent refresh races.
- A small number of `429` responses were observed under degraded load and are consistent with existing rate limiting.
```

## Suggested Clean Soak Comment

```md
Clean 4h soak run `20260313T115816Z` passed end-to-end.

- `14376/14376` requests succeeded
- `0` errors
- `0` rate-limit interference
- readiness and persistence stayed healthy
- memory artifacts were captured cleanly
- Replace optimistic locking remained correct across preflight verification
- Gemini 404 details-path bug stayed fixed across the full run
- reuse dominated: `14369` `full_plan_reuse`, `7` `fresh_generate`
```
