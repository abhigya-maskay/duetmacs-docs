---
tags: [component, cli, configuration]
aliases: [Config Loader]
---

# ConfigLoader (Skeleton for Story 001)

## Meta
- Scope date: 2025-01-08
- Author: Component Designer

## Purpose
- Define configuration constants and types to document precedence and defaults; no file I/O in Story 001.

## Responsibilities
- Define configuration search paths and precedence constants
- Define a `Config` data structure with sensible defaults
- Expose pure helpers for precedence ordering and default resolution
- Defer all file I/O and merging to a later story (Story 003)

## Boundaries
- In-scope (Story 001): Constants for search paths/precedence, `Config` type and defaults
- Out-of-scope: File reading/parsing, merging, validation, runtime updates, config writing
- Inputs (conceptual): None (pure constants/types only in Story 001)
- Outputs (conceptual): Documented precedence and default `Config`

## Collaborators & Dependencies
- Internal: None in Story 001 (no I/O)
- External: None in Story 001 (parsers and filesystem deferred)
- Notes: Document precedence — flags > env > project `.duet-rpc.toml` > user `~/.config/duet-rpc/config.toml` > defaults. Actual loading/merging and validation are deferred to Story 003.

## Risks & Open Questions
- Decision: Parsing format and libraries are deferred to Story 003.
- Decision: Filenames/paths — `.duet-rpc.toml` (project), `~/.config/duet-rpc/config.toml` (user) documented for future use.
- Risk: Schema evolution/backwards compatibility — will be handled alongside validation in Story 003.
