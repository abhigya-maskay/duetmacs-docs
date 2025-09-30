---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-005]
---

# ADR-005: Error Handling

**Status**: Accepted  
**Date**: 2025-01-05

## Context
- Need consistent, explicit error handling throughout codebase
- Haskell offers multiple error handling strategies
- Want to avoid hidden exceptions and runtime surprises

## Options
- A: Exceptions for all errors
- B: Either/ExceptT for expected errors
- C: Typed errors with error ADTs everywhere
- D: Mix: Either for domain, exceptions for IO

## Trade-offs
- A: ✓ Simple ✗ Hidden control flow
- B: ✓ Explicit expected errors ✗ Exceptions still possible
- C: ✓ Fully explicit, type-safe ✗ Verbose
- D: ✓ Pragmatic ✗ Inconsistent

## Decision Scope
- In scope: Error handling strategy, error types
- Out of scope: Error message formatting

## Decision
Typed errors with ADTs everywhere (C)

## Consequences
- Positive: No hidden exceptions, explicit error handling
- Negative: More verbose, IO operations need wrapping
- Follow-up: Define error ADTs per module

## Open Questions
None

## References
None.
