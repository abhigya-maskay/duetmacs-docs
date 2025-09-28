---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-004]
---

# ADR-004: Core Libraries

**Status**: Accepted  
**Date**: 2025-01-05

## Context
- Need robust libraries for CLI parsing, JSON, logging, HTTP
- Consistency and maintainability important


## Options
Multiple library choices per category (see details below)

## Trade-offs
- CLI: optparse-applicative for composability
- JSON: aeson + deriving-aeson for less boilerplate
- Logging: katip for structured logging
- HTTP: req for balance of safety and simplicity
- Concurrency: async + STM for standard patterns

## Decision Scope
- In scope: Core library selection
- Out of scope: Version pinning

## Decision
- CLI: optparse-applicative
- JSON: aeson + deriving-aeson
- Logging: katip
- HTTP: req
- Concurrency: async + STM
- Config: tomland

## Consequences
- Positive: Well-maintained, standard libraries
- Negative: Multiple dependencies to manage
- Follow-up: Setup proper bounds in cabal file

## Open Questions
None

## References
- All Epic 1 stories
