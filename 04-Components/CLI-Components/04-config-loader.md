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
- Leave file I/O and merging unimplemented; focus on documenting precedence for Story 001
- Verification: Unit coverage in `test/Duet/Rpc/Test/Unit/ConfigLoader.hs` guards search-order constants and defaults

## Boundaries
- In-scope (Story 001): Constants for search paths/precedence, `Config` type and defaults
- Out-of-scope: File reading/parsing, merging, validation, runtime updates, config writing
- Inputs (conceptual): None (pure constants/types only in Story 001)
- Outputs (conceptual): Documented precedence and default `Config`

## Collaborators & Dependencies
- Internal: None in Story 001 (no I/O)
- External: None in Story 001 (parsers and filesystem not implemented)
- Notes: Document precedence — flags > env > project `.duet-rpc.toml` > user `~/.config/duet-rpc/config.toml` > defaults. Actual loading/merging and validation will arrive when the loader grows beyond the skeleton.

## Risks & Open Questions
- Decision: Parsing format and libraries are intentionally unspecified in this skeleton.
- Decision: Filenames/paths — `.duet-rpc.toml` (project), `~/.config/duet-rpc/config.toml` (user) documented for eventual use.
- Risk: Schema evolution/backwards compatibility — mitigation captured for the config loading implementation phase.
