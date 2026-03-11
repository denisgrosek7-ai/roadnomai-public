# roadnomai.app

RoadNomai is a trip-planning platform with a Flutter client and a backend that coordinates plan generation, destination enrichment, transfer hints, caching, and operational observability.

## VAL 8.0: Production Hardening and Observability

VAL 8.0 moves the backend stack from a local file-oriented prototype model toward a multi-instance architecture with shared persistence, bounded degradation behavior, and first-class monitoring.

### Highlights

#### Generation memory and anti-repeat

- Added shared generation memory with:
  - global inventory reuse for compatible trip criteria
  - per-user and per-device anti-repeat history
- Reuse decisions now distinguish between:
  - `full_plan_reuse`
  - `destination_pool_recompose`
  - `fresh_generate`
- Metrics expose lookup volume, reuse ratios, novelty signals, and modeled cost-avoidance counters.

#### Postgres plus Redis runtime model

- Postgres is the durable source of truth for generation inventory and user history.
- Redis remains optional and is used for cache and coordination acceleration.
- When Redis is unavailable, the backend degrades safely and continues operating via Postgres-backed paths.
- Readiness now distinguishes between:
  - Postgres hard failure
  - Redis degraded-but-available service state

#### Observability as code

- Added `/metrics` Prometheus exposition for generation, infra, replace, readiness, and LLM pipeline behavior.
- Added:
  - Prometheus scrape configuration
  - alert rules
  - Grafana datasource provisioning
  - Grafana dashboards for business savings, infra health, and app reliability

### What this release improves

| Area | Status | Summary |
| --- | --- | --- |
| Trip generation | `hardened` | stronger orchestration, metrics, and shared-state reuse paths |
| Replace flow | `bounded` | explicit exhaustion path and safer failure handling |
| Generation memory | `active` | shared inventory, anti-repeat history, Postgres and file adapters |
| Readiness | `hardened` | `POSTGRES_UNAVAILABLE` vs `REDIS_DEGRADED` semantics |
| Observability | `active` | dashboards, alerts, metrics, and verification commands |

Sensitive provider keys are backend-only.

## Platform Concepts

### Handshake manifest

RoadNomai uses a backend-driven manifest model to coordinate app compatibility, cache invalidation, and category availability. The public-facing concept is stable: the backend can signal minimum supported versions, recommended upgrades, and category-level feature availability without shipping client secrets.

### Plugin sandboxing

Category plugins are treated as sandboxed client extension points. The public contract is that plugins stay isolated from direct network and secret-bearing backend integrations, with enforcement handled through repository tooling and linting.

## Documentation Map

- Public release notes: [docs/VAL_8_0_PUBLIC_RELEASE.md](/Users/denisgrosek/RoadNomai/docs/VAL_8_0_PUBLIC_RELEASE.md)
- Contributor setup and engineering docs: [docs/DEVELOPER_GUIDE.md](/Users/denisgrosek/RoadNomai/docs/DEVELOPER_GUIDE.md)
- Backend observability reference: [backend/docs/observability.md](/Users/denisgrosek/RoadNomai/backend/docs/observability.md)

## Getting Started

Product overview and release notes live in this README. Local development, monitoring bootstrap, manifest details, and contributor workflows are documented in the developer guide.
