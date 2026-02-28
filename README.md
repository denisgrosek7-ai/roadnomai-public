# roadnomai.app

A new Flutter project.

## Secure API Setup

Sensitive provider keys are now backend-only.

### 1) Configure backend proxy

```bash
cd backend
cp .env.example .env
# fill BACKEND_SHARED_TOKEN, GOOGLE_MAPS_SERVER_API_KEY, GEMINI_API_KEY, OPENAI_API_KEY
npm install
npm start
```

Backend now also serves transfer metadata at:

`GET /v1/transfer-info?fromLat=..&fromLng=..&toLat=..&toLng=..`

### VAL 7.4 Dynamic Context (Google Weather)

Backend can attach weather context (soft-fail) to improve trip generation decisions.

- Required ENV: `GOOGLE_WEATHER_API_KEY`
- Optional ENV: `WEATHER_PER_CALL_TIMEOUT_MS` (default `500`, hard-capped at `500`)
- Key policy:
  - DEV: application restriction `None` + API restriction `Weather API` only
  - PROD: IP-restricted egress + API restriction `Weather API` only
- Never ship this key in iOS/Android or Flutter client code; backend-only.

### 2) Configure Flutter app

```bash
cp .env.example .env
# fill ROADNOMAI_BACKEND_TOKEN
```

API base URL now supports `--dart-define`:

```bash
flutter run --dart-define=API_BASE_URL=http://localhost:8787
```

iOS simulator cannot use `localhost` to reach your Mac host process. Use your Mac LAN IP or hostname:

```bash
flutter run --dart-define=API_BASE_URL=http://<YOUR_MAC_IP>:8787
# or
flutter run --dart-define=API_BASE_URL=http://<your-hostname>.local:8787
```

Production example:

```bash
flutter run --release --dart-define=API_BASE_URL=https://api.your-domain.com
```

For iOS Google Maps SDK key, keep using `ios/Flutter/Secrets.xcconfig`.

## Transfer Hint Check

After `backend` and Flutter app are running, create a Search A plan with island hops (example: Philippines, Manila -> Negros -> Dumaguete).  
Between destination cards you should see transfer hints such as `Drive`, `Ferry`, `Flight`, or combined labels like `Ferry • Flight`.

## One-command Local Dev

From repo root:

```bash
npm run dev
```

This script starts backend (if not already running), waits for health check on `http://localhost:8787/healthz`, and then runs Flutter on `iPhone 16e`.

Optional device override:

```bash
ROADNOMAI_DEVICE="iPhone 16e" npm run dev
```

## Secret Safety Guard

Install repository git hooks once:

```bash
./scripts/install_git_hooks.sh
```

This enables `.githooks/pre-commit`, which blocks staged commits containing obvious API-key patterns.

## Plugin Sandbox Rules

Category plugins must live under `lib/registry/plugins/`.

Plugins are pure Dart extension points and must not import IO/network/internal API clients:

- `dart:io`, `dart:html`, `dart:js`, `dart:indexed_db`
- `package:http*`, `package:dio*`, `package:web_socket_channel*`
- internal proxy/client imports such as `secure_backend_proxy`

CI enforces this with custom lint rule `no_plugin_io_imports`.

## Handshake Manifest (VAL 4.1)

### Manifest endpoint

- Path: `GET /v1/manifest`
- Response contract (exact fields):
  - `min_supported_app_version` (`string`, semver)
  - `recommended_app_version` (`string`, semver)
  - `cache_epoch` (`int`)
  - `active_categories` (`array` of `{ id: string, enabled: bool }`)
  - `emergency_message` (`string | null`)

Example:

```json
{
  "min_supported_app_version": "1.0.0",
  "recommended_app_version": "1.2.0",
  "cache_epoch": 1,
  "active_categories": [
    { "id": "surprise_me", "enabled": true },
    { "id": "crystal_waters", "enabled": true },
    { "id": "hiking_beta", "enabled": false }
  ],
  "emergency_message": null
}
```

### Client behavior (HandshakeService)

- On startup:
  - fetch manifest from backend
  - persist manifest and timestamp in SharedPreferences:
    - `handshake.manifest.v1`
    - `handshake.last_fetch_ms.v1`
  - if fetch fails: load cached manifest and set `offlineMode=true`
- Version enforcement:
  - hard block when `appVersion < min_supported_app_version`
  - soft update prompt when `appVersion < recommended_app_version`

### Cache epoch (remote cache purge)

- `SignatureService` key material includes `epoch=<cache_epoch>`.
- Changing `cache_epoch` on backend invalidates all existing client cache entries safely (no schema changes required).

### Category availability rules

- `categoryStateFor(id)`:
  - `visible=true` only when category `id` exists in `active_categories` (when manifest is loaded)
  - `enabled` follows `active_categories[].enabled`
- Fallback when manifest is not loaded (first launch/offline):
  - safe UX fallback: locally known categories remain visible and enabled to avoid blocking startup.

### Emergency messaging

- If `emergency_message != null`, app shows maintenance overlay at root.

## VAL 4.2 — Per-Category Schema Versioning (Versioning Shield)

### Manifest contract (`active_categories` migration)

- Endpoint: `GET /v1/manifest`
- `active_categories` supports both formats (backward-compatible):

Legacy (VAL 4.1):

```json
"active_categories": ["surprise_me", "crystal_waters"]
```

VAL 4.2+:

```json
"active_categories": [
  { "id": "surprise_me", "enabled": true, "schema_version": 1 },
  { "id": "crystal_waters", "enabled": true, "schema_version": 2 }
]
```

Parsing rules:

- `String` element => `enabled=true`, `schema_version=1`
- `Object` element => `enabled` defaults to `true`, `schema_version` defaults to `1`

### Schema version bump SOP

- Any structural change to a category normalization/payload contract must bump `schema_version` in backend manifest.
- This guarantees isolated cache invalidation per category.

### Cache key isolation (`cv`)

- Signature key material includes:
  - `...|cat=<categoryId>|cv=<categorySchemaVersion>|sig=<signatureBase>`
- Guarantees:
  - bumping `schema_version` for one category invalidates only that category cache
  - other categories remain unaffected

### Startup/offline safety default

- If manifest is not loaded yet (first launch/offline race), client uses:
  - `categorySchemaVersion = 1`
- This prevents accidental cache-wide invalidation from missing handshake data.

### Example

- `surprise_me` schema `1 -> 2` makes `surprise_me` cache MISS, while `crystal_waters` stays HIT.

## Getting Started

This project is a starting point for a Flutter application.

A few resources to get you started if this is your first Flutter project:

- [Lab: Write your first Flutter app](https://docs.flutter.dev/get-started/codelab)
- [Cookbook: Useful Flutter samples](https://docs.flutter.dev/cookbook)

For help getting started with Flutter development, view the
[online documentation](https://docs.flutter.dev/), which offers tutorials,
samples, guidance on mobile development, and a full API reference.
