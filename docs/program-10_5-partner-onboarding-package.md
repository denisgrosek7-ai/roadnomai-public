# Program 3 — Phase 10.5 / Partner Onboarding Package

This document defines one narrow, repeatable onboarding path that the current repository can support without turning each partner into a custom project.

## Canonical pilot package

### Target partner profile

- Reference profile: one Android-first white-label leisure or travel-club pilot based on the existing `alpine_club` tenant shape in `backend/src/services/tenant_context_service.js`
- Delivery posture: shared Expo mobile app plus tenant-aware backend configuration
- Platform promise: Android-first with an explicit iOS caveat

### Included in the canonical pilot

- shared mobile shell with tenant-aware header title and accent
- tenant-aware category restrictions before planner execution
- partner-ready outbound routing with `booking_com` for `stay_hotel`
- fallback `reopen-last-trip` flow already runtime-backed on both iOS and Android
- existing restore/runtime carry-forward notes included in onboarding and support caveats

### Explicit exclusions

- dedicated branded binary identity per partner, including custom app name, icons, splash, package name, or bundle identifier
- verified tenant authentication or production-secure partner auth
- live premium billing, callbacks, reconciliation, or control-plane actions
- non-hotel live partner providers; `car_rental`, `travel_insurance`, and `local_experiences` remain stubbed partner-ready providers only
- any promise that iOS preferred external deep-link entry for `/saved/[id] -> /trip` is fully closed
- partner-specific native theming beyond the current title/accent plus existing theme key contract

### Success criteria

- one tenant config can be filled from a repeatable template and translated into a new registry entry without changing shared app logic
- Android onboarding can run through the current Expo/native baseline with tenant-aware branding and routing inputs
- partner scope stays inside the current shared app shell, current provider set, and current restore/runtime boundaries
- ownership for tenant config, mobile envs, partner inputs, and support caveats is explicit

### Abort boundary

Treat the pilot as out of scope and stop packaging it as a repeatable Phase 10.5 partner path if any of the following become required:

- dedicated branded binary identity or app-store packaging per partner
- verified auth boundary, commercial truth, callbacks, reconciliation, or support/control-plane actions
- custom category logic, custom routing logic, or custom provider integrations outside the current tenant registry and partner-routing contract
- a product promise that depends on iOS preferred external saved-link entry behaving the same as Android

## Shared core, partner layer, and internal/private layer

### Shared core

Keep the shared core limited to surfaces that are not partner-specific:

- trip generation, restore, open, and reopen flows
- itinerary persistence and local SQLite state
- search and discovery foundations
- analytics, telemetry, and observability foundations
- shared APIs, schemas, and route contracts
- shared outbound/provider helpers

### Partner layer

Keep the partner layer limited to bounded setup/configuration surfaces:

- `partner_manifest.partner_id`
- `partner_manifest.tenant_key`
- `partner_manifest.package_tier`
- `partner_manifest.platform_scope`
- `partner_manifest.required_core_version`
- `partner_manifest.is_active`
- `tenant_config.brand`
- `tenant_config.allowed_categories`
- `tenant_config.allowed_features`
- `tenant_config.partner_routing`
- `partner_manifest.outbound_mode`
- `partner_manifest.provider_mode`
- `partner_manifest.support_contact`
- `partner_manifest.alerting_identity`
- `partner_manifest.known_limitations_acknowledged`

### Internal/private layer

Keep the internal/private layer out of partner-facing setup and package claims:

- shared tokens, backend secrets, and partner-private credentials
- verified auth or trust internals
- premium/commercial truth, callbacks, reconciliation, and billing internals
- support-only evidence, incident notes, and runtime debugging artifacts
- internal scope/governance decisions that are not part of the partner setup contract

### Current implementation boundary

- Shared core is CODE-BACKED.
- The partner layer is partly CODE-BACKED through the tenant registry and runtime tenant context, and partly DOC-BACKED / CONFIG-BACKED through the setup artifact in this document.
- The internal/private layer remains intentionally outside the repeatable partner setup path.

## Config and manifest model

### Current source of truth

- Backend tenant registry stub in `backend/src/services/tenant_context_service.js`
- Mobile request headers from `apps/mobile-expo/src/api/client.ts`
- Mobile fallback envs from `apps/mobile-expo/src/constants/config.ts`
- Mobile runtime read path from `/v1/tenant/context` via `apps/mobile-expo/src/hooks/useTenantContextQuery.ts`

### Canonical onboarding artifact

Use [program-10_5-partner-config.example.json](/Users/denisgrosek/RoadNomai/docs/program-10_5-partner-config.example.json) as the fill-in manifest-first onboarding artifact.

- `partner_manifest` defines the bounded partner-layer contract.
- `tenant_config` is the portion translated manually into a new `TENANT_REGISTRY` entry.
- `onboarding_gate` is the minimum operational record that keeps package-fit, platform choice, activation state, and owner approval out of founder memory.

Important rules:

- `partner_manifest.is_active` defaults to `false`.
- The manifest does not activate a partner by itself.
- `partner_manifest.required_core_version` is a manual compatibility boundary, not an automated feature gate.
- `partner_manifest.outbound_mode = direct_web` is the safe default until a later affiliate mode exists and is actually approved.

### Required partner-layer inputs

- `partner_manifest.partner_id`
- `partner_manifest.tenant_key`
- `partner_manifest.package_tier`
- `partner_manifest.platform_scope`
- `partner_manifest.required_core_version`
- `partner_manifest.is_active`
- `partner_manifest.outbound_mode`
- `partner_manifest.provider_mode`
- `partner_manifest.support_contact`
- `partner_manifest.alerting_identity`
- `partner_manifest.known_limitations_acknowledged`
- `tenant_config.tenant_id`
- `tenant_config.tenant_slug`
- `tenant_config.tenant_mode`
- `tenant_config.brand.name`
- `tenant_config.brand.logo_key`
- `tenant_config.brand.accent`
- `tenant_config.brand.theme_key`
- `tenant_config.allowed_categories`
- `tenant_config.partner_routing.vertical_defaults`
- `tenant_config.partner_routing.disabled_verticals`
- `tenant_config.copy.home_title`

### Current implementation reality

| Field group | Evidence basis | Reality | Notes |
| --- | --- | --- | --- |
| `partner_manifest.partner_id`, `tenant_key`, `package_tier`, `platform_scope`, `required_core_version`, `is_active`, `support_contact`, `alerting_identity`, `known_limitations_acknowledged` | DOC-BACKED + CONFIG-BACKED | MANUAL | Canonical setup contract only; not yet runtime-enforced |
| `partner_manifest.outbound_mode`, `provider_mode` | DOC-BACKED + CODE-BACKED | MANUAL | Must map back to current runtime-safe `partner_routing` values; no runtime manifest loader exists |
| `tenant_id`, `tenant_mode`, `tenant_slug` | CODE-BACKED + CONFIG-BACKED | REUSABLE | Tenant selection and tenant response payload are real |
| `brand.name`, `brand.accent` | CODE-BACKED + RUNTIME-BACKED | CONFIGURABLE | Mobile header title and accent are applied in `_layout.tsx` |
| `allowed_categories` | CODE-BACKED + TEST-BACKED | CONFIGURABLE | Backend rejects disallowed categories before planner execution |
| `partner_routing` | CODE-BACKED + TEST-BACKED | CONFIGURABLE | Outbound provider defaults and disabled verticals are enforced |
| `brand.theme_key`, `brand.logo_key`, `copy.home_title`, `allowed_features`, `default_flow`, `feature_toggles` | CODE-BACKED | MANUAL | Contract-shaped and returned by the backend, but only a subset is consumed in the current mobile UI |
| native app identity (`expo.name`, `slug`, Android package, iOS bundle id) | CONFIG-BACKED | MANUAL | Exists in Expo/native config, but changing it per partner would make the pilot custom-project work |

### Validation and defaulting model

- JSON shape can be validated syntactically before review.
- Category values are validated manually against the canonical backend registry keys in `backend/src/services/category_registry.js`.
- Runtime validation exists only after `tenant_config` is translated into `TENANT_REGISTRY` and returned through `/v1/tenant/context`.
- Missing runtime validation for `partner_manifest` fields must not be treated as automation proof.
- Safe defaults:
  - `is_active = false`
  - `platform_scope = android_only`
  - `outbound_mode = direct_web`
  - non-hotel provider posture remains stubbed unless the existing partner-routing contract says otherwise

## Package-fit decision rule

Apply this table before technical onboarding starts. The result must be written into `onboarding_gate.package_fit_outcome`.

| Intake condition | Outcome | Operational rule |
| --- | --- | --- |
| Shared Expo shell, Android included, existing provider set only, chosen categories all come from the current canonical category registry, and no request asks for custom auth/commercial/control-plane behavior | PASS | Onboarding may continue inside the canonical package |
| iOS is requested in addition to Android, but only as a secondary platform with the preferred external saved-link caveat acknowledged | ESCALATE | Continue only after explicit owner approval is recorded in `package_signoff_owner`, `technical_owner`, and `package_go_no_go_approval` |
| Any required onboarding gate field is missing or owner assignment is incomplete | ESCALATE | Do not start technical onboarding until the gate record is complete |
| Dedicated branded binary request, custom provider request, verified auth/commercial-truth requirement, custom navigation/layout request, control-plane/reconciliation request, or required iOS parity for the constrained preferred external restore path | STOP | The request is outside the canonical package and must not be onboarded as repeatable Phase 10.5 scope |
| Any unknown, unsupported, or unmapped category key | STOP | Do not onboard until the request is reduced to current canonical keys or moved into a separate category-registry change |

## Platform rule

### Android default rule

- `platform_selection = android_only` is the canonical default.
- If Android is not included, the request is outside the canonical package.

### iOS exception rule

- iOS is never implied by default.
- iOS may be included only as an explicit exception on top of the Android baseline.
- `ios_caveat_acknowledged` must be `true` before iOS stays inside the package.
- iOS remains inside the package only when the partner accepts that preferred external `/saved/[id] -> /trip` is not promised as Android-equivalent pilot behavior.

### When iOS escalates or stops

- ESCALATE:
  - iOS is requested as a secondary platform and the caveat is acknowledged, but explicit owner approval is still pending
- STOP:
  - iOS is requested without Android
  - iOS parity is required for the constrained preferred external restore/open path
  - iOS inclusion is used to justify a dedicated binary, separate navigation branch, or parity promise outside the shared-shell package

## Category-selection rule

### Canonical source of truth

Use the current planner category registry keys in `backend/src/services/category_registry.js` as the onboarding allowlist source. The mobile Search A registry in `apps/mobile-expo/src/features/search-a/categoryRegistry.ts` must stay aligned, but onboarding acceptance is keyed from the backend registry.

Current canonical keys for this package:

- `crystal_waters`
- `into_the_wild`
- `snow_slopes`
- `urban_pulse`
- `adrenaline_boost`
- `gourmet_journeys`
- `hidden_heritage`
- `relax_wellness`

### Operational rule

- `onboarding_gate.chosen_category_keys` must be a non-empty subset of the canonical keys above.
- `tenant_config.allowed_categories` must exactly match `chosen_category_keys` unless the intentional choice is the unrestricted consumer-style empty allowlist.
- Unknown or unmapped keys force `STOP`.
- A request for a new category is outside this canonical onboarding path and requires a separate registry change.

## Onboarding checklist

### Ordered steps

1. Freeze the partner against the canonical package above.
2. Copy the canonical manifest example and fill `partner_manifest` first.
3. Keep `partner_manifest.is_active = false` while package-fit, ownership, and compatibility checks are still open.
4. Fill every required `onboarding_gate` field and compute `package_fit_outcome` from the decision table above.
5. Collect the required `tenant_config` fields from the example JSON.
6. Validate that `chosen_category_keys` are drawn only from the canonical backend registry keys listed above.
7. If `package_fit_outcome = ESCALATE`, stop technical onboarding until explicit approval is recorded.
8. If `package_fit_outcome = STOP`, do not continue inside this package.
9. Translate `tenant_config` into a new tenant entry in `backend/src/services/tenant_context_service.js`.
10. Set local mobile envs:
   - `EXPO_PUBLIC_ROADNOMAI_TENANT_ID`
   - `EXPO_PUBLIC_ROADNOMAI_CLIENT_KIND=mobile`
   - backend API base/shared token
   - iOS and Android Maps keys
11. Keep display-name/theme/accent env values aligned with the backend tenant config only as local fallbacks.
12. Regenerate native config with `npx expo prebuild --platform android --no-install` and `npx expo prebuild --platform ios --no-install` if native settings changed.
13. Verify tenant context resolves from `/v1/tenant/context`.
14. Attach partner health/alerts identity using [partner-health-model.md](/Users/denisgrosek/RoadNomai/docs/partner-health-model.md).
15. Mark the partner active only after the explicit activation conditions below are satisfied.
16. Verify Android pilot path against the current shared app shell.
17. If iOS is included, confirm the caveat acknowledgement is present before onboarding signoff.

### Ready-to-onboard gate

Before a partner is considered ready to onboard, all of the following must exist:

- completed `tenant_config`
- completed `onboarding_gate`
- `tenant_id`
- `chosen_category_keys`
- `package_fit_outcome`
- `package_signoff_owner`
- `technical_owner`
- `support_or_escalation_contact`
- `platform_selection`
- `package_go_no_go_approval`
- `activation_owner`
- `deactivation_owner`
- `activation_status`
- Android included as the canonical supported runtime
- `ios_caveat_acknowledged = true` if iOS is in scope at all

### Acceptance criteria

- `package_fit_outcome = PASS`, or `ESCALATE` is resolved by explicit approval before implementation starts
- tenant context resolves without unknown-tenant fallback or manual request hacking
- mobile header title and accent reflect the configured tenant
- `chosen_category_keys` match the canonical registry keys and the configured allowlist
- category restrictions behave according to the configured tenant
- `stay_hotel` outbound remains tenant-safe and provider-safe
- Android onboarding does not require new native framework work

## Activation, deactivation, and offboarding rule

### Default posture

- A new partner starts with `partner_manifest.is_active = false`.
- A copied manifest file alone does not activate a partner.
- Until activation is complete, the setup remains a review artifact plus a pending `TENANT_REGISTRY` change.

### Activation rule

Activation is allowed only when all of the following are true:

- `package_fit_outcome = PASS`, or an `ESCALATE` case is resolved by explicit owner approval
- `tenant_config` has been translated into `backend/src/services/tenant_context_service.js`
- `platform_scope` stays inside the canonical Android-first package
- `known_limitations_acknowledged` includes the current iOS, provenance, and Booking direct-web caveats when relevant
- the partner health identity has been recorded
- the activation owner flips `partner_manifest.is_active` to `true` and records `onboarding_gate.activation_status = active_manual_registry_enabled`

### Deactivation / offboarding rule

The current repo does not prove live partner offboarding automation.
Safe deactivation is therefore manual and must do only the minimum:

1. set `partner_manifest.is_active = false`
2. record the reason and owner in the onboarding gate or follow-up incident/support record
3. stop selecting the partner tenant in mobile envs, test runs, and onboarding flows
4. if runtime removal is required, edit only the affected tenant entry in `TENANT_REGISTRY` and do not mutate unrelated tenants or shared defaults
5. preserve support evidence, prior configs, and incident history instead of deleting them

### Offboarding boundary

- If a partner can be safely left inactive without removing the registry entry, prefer that lower-risk posture first.
- Use `consumer_default` only as the shared-shell fallback when partner-specific tenant selection is intentionally removed.
- Do not describe this as a control-plane deactivation feature; it is a manual repository/config discipline.

## Platform support matrix

| Area | Platform truth | Status | Notes |
| --- | --- | --- | --- |
| Tenant header/env wiring | SHARED | REUSABLE | Mobile client already sends tenant and client-kind headers |
| Tenant registry entry | SHARED | MANUAL | Current source of truth is a code-backed registry stub |
| Header title and accent branding | SHARED | CONFIGURABLE | Mobile consumes backend tenant config for these values |
| Category restriction and partner routing | SHARED | CONFIGURABLE | Enforced in backend services |
| Canonical partner onboarding path | ANDROID-SPECIFIC | REUSABLE | Android is the truthful pilot baseline |
| Preferred `/saved/[id] -> /trip` external entry | IOS-SPECIFIC | KNOWN LIMITATION | Still constrained for iOS native external entry |
| Fallback `reopen-last-trip` | SHARED | RUNTIME-BACKED | Already closed from earlier phases |
| Fresh non-dev provenance for `last_opened_trip_json` | CROSS-PLATFORM LIMITATION | KNOWN LIMITATION | Still not fully runtime-backed across both platforms |
| Dedicated branded binary identity | CROSS-PLATFORM LIMITATION | OUT OF SCOPE | Would require manual native package/bundle/app-name work per partner |

The canonical pilot is therefore Android-first with an iOS caveat, not a blanket cross-platform promise.

## Package and tier boundaries

| Package shape | Includes | Excludes | Current status | Notes |
| --- | --- | --- | --- | --- |
| White-label Lite | Android-first shared shell, tenant branding contract, category allowlist, existing partner routing, Booking direct-web hotel outbound | dedicated binary, verified auth/commercial truth, custom providers, control-plane actions, required iOS parity | SUPPORTED TODAY | This is the canonical package defined in this repository |
| White-label Pro | White-label Lite plus broader use of existing contract fields like `theme_key`, `logo_key`, `allowed_features`, and an optional reviewed iOS exception | dedicated binary, custom providers, verified auth/commercial truth, native theming pipeline, parity promises | STRUCTURED BUT NOT YET PROVEN | Contract fields exist, but runtime/UI consumption remains partial |
| B2B | `tenant_mode = b2b_headless`, tenant-aware request context, bounded routing/category contract | custom integration platform work, verified auth, partner-specific control plane, custom provider contracts | STRUCTURED BUT NOT YET PROVEN | Headless/code surfaces exist, but this is not the canonical repeatable package in Phase 10.5 |
| B2C | `consumer_default` shared-core experience with no partner-specific overlay | partner-specific branding, partner support promises, tenant-specific package commitments | SUPPORTED TODAY | This is the consumer/shared-core path, not a partner package |

### Package truth rules

- White-label Lite remains the only package shape that is both canonical and supportable today.
- White-label Pro must not be used to imply live custom-provider, branded-binary, or verified-auth capability.
- B2B may be described as a bounded integration posture only when the ask stays inside current tenant and routing contracts.
- B2C is the shared-core consumer path and must not be sold as disguised white-label capability.

## Branding and template boundaries

### Safe and supported in pilot phase

- tenant display name in the mobile header
- tenant accent color in the mobile header chrome
- category allowlist
- partner-routing defaults and disabled verticals

### Fixed or intentionally bounded

- shared RoadNomai app shell and shared Expo project structure
- current dark theme and current shared component styling
- current outbound provider set
- current native app identity and asset pipeline

### Explicitly unsupported in pilot phase

- per-partner icon/splash pipeline
- custom layout/system-navigation changes per tenant
- custom store/distribution package split per tenant
- partner-specific logic branches outside tenant registry and existing route/provider gating

## Support and ownership minimum

### Owner roles

- Backend / Platform owner: tenant registry entry, category allowlist, partner-routing defaults, shared token path
- Mobile / Client owner: Expo envs, Maps keys, native prebuild step, Android packaging validation
- Partner / Commercial owner: partner inputs, signoff on included vs excluded surfaces, escalation contact
- Support owner: named contact for onboarding issues and known limitation disclosure

### Minimum handoff package

- completed `partner_manifest`
- completed `tenant_config`
- completed `onboarding_gate`
- chosen tenant id and partner profile
- Android build path and local env checklist
- package tier and activation status
- current known limitations copied from the section below
- support contact and escalation contact

### What no longer depends on founder memory

- package-fit now uses an explicit PASS / ESCALATE / STOP rule
- iOS inclusion now uses an Android-default / iOS-exception rule with explicit acknowledgement
- category selection now uses an explicit backend registry allowlist source

### What still remains manual

- translating `tenant_config` into a new `TENANT_REGISTRY` entry
- collecting owner names and contacts
- recording explicit approval when an iOS exception is allowed
- flipping `is_active` only after the manual activation gate is satisfied
- keeping partner health state as a manual operational record rather than a live control plane

## Codex governance for partner-layer changes

Use [AGENTS.md](/Users/denisgrosek/RoadNomai/AGENTS.md) as the repository-level rule surface for what Codex may auto-fix versus what must stay proposal-only/report-only.

## Known limitations carried forward

- iOS preferred external deep-link/native entry for `/saved/[id] -> /trip` remains constrained and must not be promised as equivalent to Android
- fresh non-dev cross-platform provenance for `last_opened_trip_json` is still not fully runtime-backed
- verified tenant auth, provider-backed premium truth, reconciliation, and control-plane capabilities remain out of scope for this package
- non-hotel partner verticals remain stubbed partner-ready providers only
