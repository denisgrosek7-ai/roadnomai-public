# RoadNomai

RoadNomai is an AI-powered travel planning platform that creates complete, multi-destination itineraries in seconds.

Users define the number of travel days, choose one or more countries, and select how many destinations they want to visit. Based on these preferences, RoadNomai generates an optimized travel route with destination flow, estimated travel times, distances between destinations, and relevant transport connections, including flights and ferry routes for island travel.

The platform is designed to make trip planning faster, easier, and more flexible. If a destination does not fit the traveler’s preferences or has already been visited, it can be replaced with a suitable alternative without breaking the overall route logic.

RoadNomai also helps structure the stay across each stop by automatically distributing the trip based on the total number of days and selected destinations. With the **Explore Nearby** feature, users can discover local highlights around each destination, including cultural, historical, culinary, and other points of interest.

Built as a flexible platform for **white-label**, **B2B**, and **B2C** use cases, RoadNomai is suitable for individual travelers, travel agencies, and travel-focused businesses that want to offer smart, automated itinerary creation with minimal manual planning.

## Overview

RoadNomai is built as a modular travel platform with a shared core that supports intelligent trip planning and partner-facing travel experiences.

The platform is intended to support:

- **white-label deployments** for agencies and partners
- **B2B partner-aware product models**
- **B2C branded product experiences**
- modular client and backend surfaces built on a common core
- configurable branding, routing, and platform structure
- scalable and flexible itinerary generation workflows

At a high level, RoadNomai is designed around:

- a **shared product core**
- a **partner-aware / tenant-aware configuration layer**
- modular product surfaces for planning and trip-related flows
- configurable branding and theming
- public-safe schemas and platform concepts

**One platform core, multiple product directions.**

## Core Capabilities

- AI-assisted itinerary generation
- Multi-country and multi-destination trip planning
- Optimized travel routes based on user preferences
- Flight and ferry connection support
- Travel time and distance calculation between destinations
- Flexible destination replacement without disrupting route flow
- Automatic stay distribution across the itinerary
- **Explore Nearby** recommendations for each destination
- White-label, B2B, and B2C platform support

## Who It Is For

RoadNomai is designed for:

- travelers who want a faster and easier way to plan complex trips
- users who want inspiration without spending hours researching routes
- travel agencies that want to build complete travel itineraries efficiently
- businesses looking for a white-label or partner-ready travel planning solution

## Current Status

RoadNomai is currently **in active development**.

The platform is still evolving, and some features, flows, and documentation are not yet final. This repository presents the public-facing concept, platform direction, and selected documentation as the product continues to grow.
## Mobile App Status

The RoadNomai mobile app is currently being developed with **Expo / React Native**.

This repository also includes focused mobile restore and reopen test coverage for both **Android** and **iOS**, with attention to persisted trip state and reopen behavior.

Current validation includes:

- fallback reopen flow: `reopen-last-trip`
- preferred restore flow: `/saved/[id] -> /trip`
- persisted pointer behavior through `last_opened_trip_json`

For detailed technical coverage, see:
- [`docs/mobile-restore-coverage.md`](docs/mobile-restore-coverage.md)

## Repository Scope

This public repository focuses on the **public-safe platform view**, including:

- platform and architecture overviews
- public-safe concepts and schemas
- selected documentation
- contributor-facing technical context where appropriate

This repository does **not** include sensitive production details such as:

- partner-specific configuration
- internal operational runbooks
- private integrations
- production deployment internals
- internal debugging and runtime artifacts

## Summary

RoadNomai is a smart travel planning platform that transforms a few user inputs into a structured, optimized, and flexible travel experience. From route generation and transport connections to stay distribution and nearby discovery, the goal is simple:

**to make travel planning easier, faster, and more intelligent.**
