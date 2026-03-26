## Mobile App Status

The RoadNomai mobile app is currently being developed with **Expo / React Native**.

This repository also includes focused mobile restore and reopen test coverage for both **Android** and **iOS**, with special attention to persisted trip state and reopen behavior.

## Mobile Restore Coverage

The current mobile test surface is designed to validate how the app restores previously opened trips and how persisted state is handled across reopen flows.

### Key restore paths

- **Fallback reopen flow:** `reopen-last-trip`
- **Preferred restore flow:** `/saved/[id] -> /trip`

These two flows are treated separately and should not be considered equivalent.

A successful fallback reopen test does **not** automatically prove that the preferred saved-trip restore path is fully validated.

## Current Coverage

### Android
Android coverage currently verifies:

- fallback reopen from the home flow
- preferred saved-alias restore via `/saved/[id] -> /trip`
- persisted pointer updates through `last_opened_trip_json`

Primary Android runtime test:

- `apps/mobile-expo/android/app/src/androidTest/java/com/anonymous/roadnomaimobileexpo/ReopenLastTripInstrumentationTest.kt`

### iOS
iOS coverage currently verifies:

- fallback reopen behavior through the native UI flow

Primary iOS runtime test:

- `RoadNomaiMobileTests/testReopenLastTripFromHome`

> Important:
> iOS fallback reopen is currently covered.
> The preferred `/saved/[id] -> /trip` restore path should only be treated as complete on iOS when there is explicit runtime proof for that exact flow.

## Validation Principle

Restore-related behavior in this repository should be interpreted carefully:

- **runtime-backed** = proven through a real device or simulator flow
- **test-backed** = covered in repository tests
- **code-backed** = visible in code, but not yet fully runtime-proven

This distinction is important because fallback success, preferred-path success, and seeded test setup are **not interchangeable**.

## Supporting Test Surface

Additional repository-level tests support mobile restore validation, including:

- execution-layer behavior
- analytics normalization
- persisted pointer and SQLite bootstrap behavior

These tests are useful for confidence, but they do **not** replace runtime-backed proof on Android or iOS.

## Summary

RoadNomai’s mobile restore test coverage exists to answer a focused question:

**What is truly proven about restore behavior on Android and iOS, and what is still only inferred?**

This helps keep restore, reopen, and persisted trip-state behavior clear, reviewable, and technically honest while the product continues to evolve.
