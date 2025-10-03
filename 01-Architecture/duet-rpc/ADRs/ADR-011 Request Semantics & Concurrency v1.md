---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-011]
---

# ADR-011: Request Semantics & Concurrency v1

**Status**: Accepted  
**Date**: 2025-10-03

## Context
- Story 003 defines request handling behavior for Emacs clients and tests.
- [[ADR-006 State Management|ADR-006]] chooses STM for general state management but does not constrain v1 scheduling.

## Options
- Single-threaded sequential dispatcher vs. concurrent worker pool.
- Support JSON-RPC batch arrays vs. reject in v1.
- Acceptable ID types and ordering guarantees.

## Decision Scope
- In scope: Concurrency model, batching, notifications, request IDs, response ordering, initialization requirements.
- Out of scope: Transport/framing ([[ADR-010 RPC IO Framing & Stdout Discipline|ADR-010]]), error mapping ([[ADR-013 Daemon Lifecycle & Signals|ADR-013]]).

## Decision
- Execution model: Single-threaded, sequential; process and respond to one request completely before starting the next.
- Batching: Not supported; a JSON array request returns −32600 with `error.message` set to `"Batch requests not supported"`.
- Notifications: Supported; process without response. Unknown-method notifications logged at `warn`. Parameter validation or internal errors logged at `error`. Notifications do not send responses or terminate daemon.
- Request IDs: Accept `string | number | null`; echo exactly, preserving type and value. Reject `object | array | boolean` with −32600.
- Ordering: Responses are sent in the same order as requests were received (frames arrive in order per [[ADR-010 RPC IO Framing & Stdout Discipline|ADR-010]]).
- Initialization: No required handshake; all methods callable immediately.
- Client guidance: Clients SHOULD NOT pipeline concurrent requests.

## Consequences
- Simpler dispatcher and state; deterministic test behavior.
- Future concurrency requires explicit change management and documentation.

## Open Questions
None

## References
- Story 003: `03-Epics/V1-Foundation/01-Project-Bootstrap/Stories/003-json-rpc-daemon/003-json-rpc-daemon.md`
- [[ADR-006 State Management|ADR-006]]: State Management

