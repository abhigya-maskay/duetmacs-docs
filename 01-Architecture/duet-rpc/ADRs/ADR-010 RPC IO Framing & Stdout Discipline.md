---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-010]
---

# ADR-010: RPC I/O Framing & Stdout Discipline

**Status**: Accepted  
**Date**: 2025-10-03

## Context
- [[ADR-002 RPC Protocol|ADR-002]] selected JSON-RPC 2.0 with LSP-style framing; header, encoding, and stdout rules unspecified.
- Story 003 specifies behavior for Emacs interop and reliable tests.
- Must forbid non-JSON on stdout; clients parse only framed responses.

## Options
- Framing: LSP-style length-prefix vs. newline-delimited JSON.
- Content-Type policy: strict `application/vscode-jsonrpc` vs. permissive `application/json`.
- Encoding: UTF-8 only vs. multiple charsets.
- Stdout policy: stdout-only JSON vs. mixed logs and JSON.

## Decision Scope
- In scope: Wire framing, headers, encoding, size/timeouts, stdout/stderr policy.
- Out of scope: Method semantics, concurrency, ID rules ([[ADR-011 Request Semantics & Concurrency v1|ADR-011]]), error mapping ([[ADR-013 Daemon Lifecycle & Signals|ADR-013]]).

## Decision
- Transport: JSON-RPC 2.0 over stdio with LSP-style framing (`Content-Length: N\r\n\r\n{json}`).
- Encoding: UTF-8 only.
- Headers:
  - Required: `Content-Length` equals UTF-8 byte length of body.
  - Optional: `Content-Type: application/vscode-jsonrpc; charset=utf-8` (charset case-insensitive; parameter order not significant).
    - Wrong media type or charset returns error −32600.
  - Unknown headers (other than Content-Type): ignored.
  - Header names are case-insensitive.
- Limits & timeouts:
  - Max request size: 10 MB. Content-Length > 10 MB → respond −32600 with `id: null`, discard payload without reading.
  - Read timeout: 30s per framed message. On timeout: discard partial, log, continue (connection remains open; consecutive timeouts do not terminate daemon).
  - Content-Length mismatch (EOF before N bytes or excess after) → −32700; do not parse partial payloads.
- Process multiple frames in arrival order ([[ADR-011 Request Semantics & Concurrency v1|ADR-011]] for response ordering).
- Stdout discipline: stdout carries only framed JSON-RPC responses; logs/diagnostics to stderr or `DUET_RPC_LOG`.
- Compatible with Emacs `jsonrpc.el`.

## Consequences
- Parser/IO loop enforces limits and headers.
- Tests must cover framing correctness, header variants, oversize, timeout, and length mismatch.
- Logger must never write to stdout.

## Open Questions
None

## References
- Story 003: `03-Epics/V1-Foundation/01-Project-Bootstrap/Stories/003-json-rpc-daemon/003-json-rpc-daemon.md`
- [[ADR-002 RPC Protocol|ADR-002]]: RPC Protocol

