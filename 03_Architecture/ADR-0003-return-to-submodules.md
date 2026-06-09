# ADR-0003: Return to Submodules

## Status

Accepted

## Date

2026-06-09

## Context

The workspace briefly moved `frontend/` and `knowledge-base/` into the parent repository to simplify development.

The project direction changed again: frontend, backend, and knowledge base should keep separate Git histories while the parent workspace coordinates their selected versions.

## Decision

Track these directories as Git submodules in the parent Fortis workspace:

- `frontend/` from `Ramilko37/fortis-front`
- `backend/` from `Ramilko37/fortis-back`
- `knowledge-base/` from `Ramilko37/fortis-knowledge-base`

The parent repository stores workspace-level instructions and submodule pointers.

## Consequences

- Contributors must initialize submodules after cloning the parent workspace.
- Changes inside `frontend/`, `backend/`, or `knowledge-base/` must be committed and pushed in the child repository first.
- The parent workspace must then commit the updated submodule pointer.
- Frontend, backend, and knowledge-base histories remain independent.
