# RoadNomai

RoadNomai is a modular travel platform designed to support **white-label**, **B2B**, and **B2C** product models from a shared core.

## Overview

RoadNomai is built as a shared platform foundation for travel planning and related partner-facing experiences.

The platform is intended to support:

- **white-label deployments** for agencies and partners
- **B2B partner-aware product models**
- **B2C branded product experiences**
- modular client and backend surfaces built on a common core
- configurable branding, routing, and platform posture
- observability and verification-oriented engineering workflows

This public repository focuses on the **public-safe platform view**:
architecture, schemas, platform concepts, and selected documentation.

Sensitive production details, partner-specific configuration, operational runbooks, and internal verification artifacts remain private.

---

## What RoadNomai Supports

### White-label

RoadNomai is designed to support partner-branded deployments from a shared platform core, with configurable branding, routing posture, and feature exposure.

### B2B

The platform supports a partner-aware model for agency and integration-oriented use cases, where one core can serve multiple deployment contexts through controlled configuration.

### B2C

The same platform foundation can also support direct branded consumer-facing flows without splitting the product into separate codebases.

---

## Platform Model

At a high level, RoadNomai is structured around:

- a **shared product core**
- a **partner-aware / tenant-aware configuration layer**
- modular product surfaces for planning, trip handling, and related flows
- configurable branding and theming
- public-safe schemas and platform contracts
- observability and verification discipline

The public-facing architecture goal is simple:

**one platform core, multiple product directions.**

---

## Repository Scope

This public repository includes:

- public-safe documentation
- platform and architecture overviews
- selected schemas and public-facing contracts
- sanitized product and engineering documentation
- contributor-facing setup material where appropriate

This public repository intentionally does **not** include:

- partner-specific configuration
- production deployment internals
- sensitive provider or billing logic
- support/control-plane internals
- internal alert routing
- incident playbooks or escalation procedures
- internal runtime/debug evidence
- private operational governance artifacts

---

## Documentation Map

### Core public docs

- [`docs/platform-overview.md`](docs/platform-overview.md)
- [`docs/architecture-overview.md`](docs/architecture-overview.md)
- [`docs/white-label-overview.md`](docs/white-label-overview.md)
- [`docs/b2b-overview.md`](docs/b2b-overview.md)
- [`docs/b2c-overview.md`](docs/b2c-overview.md)
- [`docs/feature-matrix.md`](docs/feature-matrix.md)
- [`docs/public-status.md`](docs/public-status.md)
- [`docs/deployment-boundaries.md`](docs/deployment-boundaries.md)

### Public technical references

- [`docs/DEVELOPER_GUIDE.md`](docs/DEVELOPER_GUIDE.md)
- [`backend/docs/observability.md`](backend/docs/observability.md)

### Release-oriented public notes

- [`docs/VAL_8_0_PUBLIC_RELEASE.md`](docs/VAL_8_0_PUBLIC_RELEASE.md)

---

## Current Public Position

RoadNomai should be understood publicly as:

- a **shared platform core**
- a **white-label-capable architecture**
- a **B2B/B2C-capable product foundation**
- a system with a strong emphasis on **modularity**, **configurability**, and **verification discipline**

The public repository is meant to communicate platform direction and public-safe engineering concepts, not to expose sensitive production internals.

---

## Getting Started

Use this repository as a public-facing reference for:

- platform structure
- high-level architecture
- public-safe concepts and schemas
- contributor-facing engineering context

If you are looking for partner provisioning, production operations, private integrations, or internal runtime/debug workflows, those remain intentionally outside the public scope.
