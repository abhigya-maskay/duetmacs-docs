---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-010]
---

# ADR-010: RPC I/O Framing & Stdout Discipline

**Status**: Accepted  
**Date**: 2025-10-03

## Context
- ADR-002 selected JSON-RPC 2.0 with LSP-style framing but did not fix header, encoding, and stdout rules.
- Story 003 specifies precise behavior needed for Emacs interop and reliable tests.
- We must forbid non-JSON output on stdout to avoid breaking clients that parse framed responses.

## Options
- Framing: LSP-style length-prefix vs. newline-delimited JSON.
- Content-Type policy: strict `application/vscode-jsonrpc` vs. permissive `application/json`.
- Encoding: UTF-8 only vs. multiple charsets.
- Stdout policy: stdout-only JSON vs. mixed logs and JSON.

## Decision Scope
- In scope: Wire framing, headers, encoding, size/timeouts, stdout/stderr policy.
- Out of scope: Method semantics, concurrency, ID rules (see ADR-011), detailed error mapping (see ADR-013).

## Decision
- Transport: JSON-RPC 2.0 over stdio with LSP-style framing (`Content-Length: N\r\n\r\n{json}`).
- Encoding: UTF-8 only.
- Headers:
  - Required: `Content-Length` equals UTF-8 byte length of body.
  - Optional: `Content-Type: application/vscode-jsonrpc; charset=utf-8` (charset case-insensitive; parameter order not significant).
  - Unknown headers: ignored.
- Limits & timeouts:
  - Max request size: 10 MB → respond −32600 and discard payload.
  - Read timeout: 30s for a complete framed message; on timeout discard partial, log, continue.
  - Content-Length mismatch → −32700.
- Multiple back-to-back frames are valid and must be processed in arrival order.
- Stdout discipline: stdout carries only framed JSON-RPC responses; logs/diagnostics go to stderr or to the path in `DUET_RPC_LOG`.
- Compatibility: Maintain compatibility with Emacs `jsonrpc.el`.

## Consequences
- The parser/IO loop enforces limits and header checks, improving robustness.
- Tests must cover framing correctness, header variants, oversize, timeout, and length mismatch.
- Logger configuration must never write to stdout.

## Open Questions
None

## References
- Story 003: `03-Epics/V1-Foundation/01-Project-Bootstrap/Stories/003-json-rpc-daemon/003-json-rpc-daemon.md`
- ADR-002: RPC Protocol

