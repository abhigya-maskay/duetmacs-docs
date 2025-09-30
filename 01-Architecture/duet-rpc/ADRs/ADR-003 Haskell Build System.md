---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-003]
---

# ADR-003: Haskell Build System

**Status**: Accepted  
**Date**: 2025-01-05

## Context
- Building a production CLI tool in Haskell
- Need reproducible builds and dependency management
- Developer experience matters

## Options
- A: Stack (curated package sets)
- B: Cabal (official, flexible)
- C: Nix (reproducible)
- D: Bazel (powerful)

## Trade-offs
- A: ✓ Reproducible, beginner-friendly ✗ Less flexible
- B: ✓ Flexible, official ✗ Dependency conflicts
- C: ✓ Perfect reproducibility ✗ Complex, niche
- D: ✓ Scalable ✗ Steep learning curve

## Decision Scope
- In scope: Build system, dependency management
- Out of scope: CI/CD pipeline

## Decision
Cabal (B) with GHC 9.10

## Consequences
- Positive: Official tooling, maximum flexibility
- Negative: Need careful dependency management
- Follow-up: Setup freeze files for reproducibility

## Open Questions
None

## References
- Story 001: CLI Version/Help Bootstrap

