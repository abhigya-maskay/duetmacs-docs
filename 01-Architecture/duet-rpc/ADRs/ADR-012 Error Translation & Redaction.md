---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-012]
---

# ADR-012: Error Translation & Redaction

**Status**: Accepted  
**Date**: 2025-10-03

## Context
- [[ADR-005 Error Handling|ADR-005]] mandates typed errors with ADTs internally.
- Story 003 requires predictable JSON-RPC error codes and safe messages.

## Options
- Minimal JSON-RPC messages vs. verbose details.
- Include rich `error.data` vs. small machine-readable hints only.
- Logging severity policies for client-caused vs. server-caused issues.

## Decision Scope
- In scope: Code mapping, response messages, `error.data` shape, redaction and logging guidance.
- Out of scope: Transport/framing ([[ADR-010 RPC IO Framing & Stdout Discipline|ADR-010]]), dispatcher ([[ADR-011 Request Semantics & Concurrency v1|ADR-011]]).

## Decision
- Canonical mapping ([[ADR-010 RPC IO Framing & Stdout Discipline|ADR-010]] for framing errors):
  - −32700 Parse Error → invalid JSON or Content-Length mismatch.
  - −32600 Invalid Request → malformed JSON-RPC envelope, unsupported Content-Type/charset, request size > 10 MB, invalid id type, batch array.
  - −32601 Method Not Found → unknown method.
  - −32602 Invalid Params → wrong/missing params.
  - −32603 Internal Error → unexpected server faults; daemon stays alive.
- Error response `id`: Echo request `id` exactly when parseable; use `id: null` if unparseable or size exceeded before reading.
- Response messages: Use canonical JSON-RPC 2.0 error messages ("Parse error", "Invalid Request", "Method not found", "Invalid params", "Internal error") with clarifications when needed.
- `error.data` (minimal, non-sensitive):
  - For −32602: `{ "param": "...", "expected": "...", "received": "...", "accepted": ["..."]? }`.
  - For −32600: `{ "reason": "unsupported-content-type|oversize|bad-charset|invalid-id-type|batch-not-supported" }`.
  - Never include stack traces, filesystem paths, tokens, or environment values.
- Logging vs. client payloads:
  - Client responses remain minimal; detailed diagnostics to logs with request id correlation when available.
  - Redact tokens/paths; never log payload bodies for parse/oversize errors.

## Consequences
- Predictable client behavior with safe, non-leaky errors.
- Centralized error mapping simplifies testing and consistency.

## Open Questions
None

## References
- Story 003: `03-Epics/V1-Foundation/01-Project-Bootstrap/Stories/003-json-rpc-daemon/003-json-rpc-daemon.md`
- [[ADR-005 Error Handling|ADR-005]]: Error Handling

