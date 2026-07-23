# ADR-0012: Fortis Design System Foundations

- **Date:** 2026-07-22
- **Status:** accepted

## Context

Open Design produced the Fortis UI Kit v1.0 Final as an implementation-ready design contract. The source project is private and sits outside Git, while the frontend currently contains separate dashboard, prototype, shadcn/Radix and Ant Design styling layers. A global token replacement or a wholesale component migration would risk the working GIS flow.

## Decision

- The versioned handoff lives in `frontend/docs/design-system/fortis-ui-kit-v1.0/`.
- Shared runtime primitives live in `frontend/src/shared/ui/fortis/`, in line with [[ADR-0006-frontend-refactoring-boundaries]].
- Fortis tokens are provider-scoped through `data-fortis-theme` and `data-fortis-density`; they do not replace dashboard or prototype tokens globally.
- New Fortis icon-only UI resolves semantic names through a typed Lucide wrapper. Existing Ant Icons remain supported only as staged migration inputs.
- The existing Manrope font variable is the safe sans fallback until IBM Plex Sans has a locally approved loading strategy. IBM Plex Mono continues to support data/version values.
- No Storybook, overlay-provider migration, GIS Workspace composition, offline persistence, or global UI migration is introduced by the foundations phase.

## Consequences

- The handoff becomes durable project memory instead of depending on private `.od` storage.
- Existing routes retain their current rendering and behavior while new Fortis primitives can be piloted incrementally.
- Future work must integrate real MapLibre/deck.gl and project store state rather than reusing prototype HTML as runtime UI.
- Overlay focus management, browser zoom, screen readers, and visual-regression fixtures remain explicit verification tasks before a production migration.
