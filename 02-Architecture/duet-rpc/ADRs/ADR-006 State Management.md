---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-006]
---

# ADR-006: State Management

**Status**: Accepted  
**Date**: 2025-01-05

## Context
- Daemon needs to manage concurrent operations safely
- Session state, connections, and streaming responses
- Haskell offers STM for safe concurrent state

## Options
- A: STM with TVar/TMVar
- B: ReaderT pattern with IORef
- C: State monad transformer
- D: Message-passing only

## Trade-offs
- A: ✓ Composable, deadlock-free ✗ Learning curve
- B: ✓ Simple ✗ Race conditions possible
- C: ✓ Pure interface ✗ Not concurrent-safe
- D: ✓ Actor-like ✗ Complex for simple state

## Decision Scope
- In scope: Runtime state management
- Out of scope: Persistence

## Decision
STM with TVar/TMVar (A)

## Consequences
- Positive: Safe concurrent access, composable
- Negative: STM learning curve
- Follow-up: Design state types carefully

## Open Questions
None

## References
- Story 005: Emacs Subprocess Management

