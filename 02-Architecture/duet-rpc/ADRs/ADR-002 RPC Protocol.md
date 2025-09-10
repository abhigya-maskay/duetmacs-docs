---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-002]
---

# ADR-002: RPC Protocol

**Status**: Accepted  
**Date**: 2025-01-05

## Context
- Emacs needs bidirectional communication with daemon
- Must support request/response and streaming
- Protocol should be well-specified and tooling-friendly

## Options
- A: JSON-RPC 2.0 over stdio
- B: Custom line-delimited JSON
- C: MessagePack RPC
- D: gRPC

## Trade-offs
- A: ✓ Standard, good tooling ✗ Verbose
- B: ✓ Simple ✗ No standard, custom parsing
- C: ✓ Efficient ✗ Less tooling, binary format
- D: ✓ Powerful ✗ Complex, needs code generation

## Decision Scope
- In scope: Wire protocol, message format
- Out of scope: Transport (using stdio)

## Decision
JSON-RPC 2.0 (A) with LSP-style length-prefixed framing

## Consequences
- Positive: Well-specified, extensive tooling, Emacs support
- Negative: JSON overhead for large payloads
- Follow-up: Implement streaming via notifications

## Open Questions
None

## References
- Story 004: CLI RPC Ping
- Story 005: Emacs Subprocess Management

