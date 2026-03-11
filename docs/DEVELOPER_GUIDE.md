# RoadNomai Developer Guide

This document is intended for public repository contributors. It covers local setup, development workflows, monitoring bootstrap, and engineering-facing integration concepts that do not belong in the public product-facing README.

Operational security procedures, internal thresholds, incident playbooks, and abuse-sensitive implementation details are intentionally excluded.

## Local Backend Setup

```bash
cd backend
cp .env.example .env
npm install
npm start
```

Sensitive provider keys stay backend-only. Do not ship provider credentials in Flutter client code.

## Local Flutter Setup

```bash
cp .env.example .env
flutter run --dart-define=API_BASE_URL=http://localhost:8787
```

If your simulator or device cannot resolve `localhost` to the host machine, use a reachable host IP or hostname instead:

```bash
flutter run --dart-define=API_BASE_URL=http://<HOST_IP>:8787
```

For iOS Google Maps SDK configuration, continue using `ios/Flutter/Secrets.xcconfig`.

## One-Command Local Dev

From the repository root:

```bash
npm run dev
```

This starts the backend when needed, waits for a healthy local response, and then launches the mobile app workflow.

## Transfer Hint Check

After backend and mobile app are running, create a trip with island or long-distance hops. The UI should show transfer hints such as `Drive`, `Ferry`, `Flight`, or combined labels like `Ferry • Flight` between destination cards.

## Monitoring Stack

Local monitoring bootstrap:

```bash
PG_PORT=5433 REDIS_PORT=6380 docker compose --profile dev --profile monitoring up -d \
  postgres redis backend-migrate backend backend-2 prometheus grafana
```

Useful local endpoints:

- Grafana: `http://localhost:3000`
- Prometheus: `http://localhost:9090`
- Backend readiness: `http://localhost:8788/readyz`
- Backend metrics: `http://localhost:8788/metrics`

More detail is documented in [backend/docs/observability.md](/Users/denisgrosek/RoadNomai/backend/docs/observability.md).

## Secret Safety Guard

Install repository git hooks once:

```bash
./scripts/install_git_hooks.sh
```

This enables the pre-commit checks that block obvious secret patterns in staged changes.

## Plugin Sandbox Rules

Category plugins must live under `lib/registry/plugins/`.

Plugins are pure Dart extension points and must not import:

- `dart:io`, `dart:html`, `dart:js`, `dart:indexed_db`
- `package:http*`, `package:dio*`, `package:web_socket_channel*`
- internal proxy or secret-bearing backend client modules

Repository linting enforces these boundaries.

## Handshake Manifest

RoadNomai uses a backend manifest to coordinate:

- minimum and recommended app versions
- cache invalidation epochs
- category visibility and enablement
- maintenance or emergency messaging

On startup, the client fetches the manifest, caches it locally, and falls back to the cached copy when offline. Version enforcement supports both hard minimum-version blocking and softer upgrade nudges.

## Per-Category Schema Versioning

Category configuration supports per-category schema versioning so cache invalidation can stay isolated to the affected category instead of forcing a global cache purge.

Contributor rule:

- if a category payload or normalization contract changes structurally, bump that category's schema version in the manifest-producing backend path

This preserves compatibility while minimizing unnecessary cache churn across unrelated categories.
