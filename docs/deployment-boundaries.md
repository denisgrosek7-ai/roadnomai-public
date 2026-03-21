# Deployment Boundaries

RoadNomai separates the public-safe platform view from private operational deployment details.

## Public Boundary

The public repository is appropriate for:

- architectural understanding
- public-safe documentation
- selected contributor-facing setup context
- public release communication

## Private Boundary

The following remain intentionally private:

- partner-specific deployment configuration
- production routing and support internals
- sensitive provider or billing logic
- private incident procedures
- private control-plane or manual-action flows
- restricted operational governance artifacts

## Why This Boundary Exists

The purpose of the boundary is to keep the repository useful and professional in public while avoiding exposure of sensitive operational details.

## Practical Reading Rule

If a topic would expose:

- private partner identity
- sensitive runtime controls
- internal operational decisions
- non-public commercial behavior

it should be treated as outside the public repository scope.
