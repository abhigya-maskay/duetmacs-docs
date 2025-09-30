---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-009]
---

# ADR-009: Testing Strategy

**Status**: Accepted  
**Date**: 2025-01-05

## Context
- Need reliable testing for RPC protocol and business logic
- Property testing valuable for parsers/serializers
- Integration testing critical for Emacs interop

## Options
- A: Primarily unit tests
- B: Mix of unit and property
- C: Property for core, unit for integration
- D: Minimal testing

## Trade-offs
- A: ✓ Simple, specific ✗ Missing edge cases
- B: ✓ Balanced ✗ More complex
- C: ✓ Targeted approach ✗ Need both frameworks
- D: ✓ Fast development ✗ Poor quality

## Decision Scope
- In scope: Test strategy, frameworks
- Out of scope: Coverage targets

## Decision
Property tests for core logic, unit for integration (C) using tasty framework

## Consequences
- Positive: Good coverage, catches edge cases
- Negative: Two testing styles to maintain
- Follow-up: Setup tasty with QuickCheck integration

## Open Questions
None

## References
None.
