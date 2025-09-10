---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-001]
---

# ADR-001: Architecture Style

**Status**: Accepted  
**Date**: 2025-01-05

## Context
- Need to support both Emacs subprocess integration and CLI one-shot commands
- Startup latency matters for Emacs UX
- Must handle long-running AI streaming operations

## Options
- A: Simple CLI tool (stateless, per-command execution)
- B: Client-server (persistent daemon, RPC protocol)  
- C: Hybrid (daemon mode + one-shot commands)
- D: Microservices (separate services)

## Trade-offs
- A: ✓ Simple ✗ High startup latency for Emacs
- B: ✓ Low latency ✗ No CLI one-shot support
- C: ✓ Best of both ✗ More complex architecture
- D: ✓ Scalable ✗ Overkill for single-user tool

## Decision Scope
- In scope: Process model, communication patterns
- Out of scope: Deployment, packaging

## Decision
Hybrid model (C) - daemon for Emacs, one-shot commands for CLI

## Consequences
- Positive: Low latency for Emacs, scriptable CLI
- Negative: Two code paths to maintain
- Follow-up: Ensure shared business logic between modes

## Open Questions
None

## References
- Epic: Project Bootstrap

