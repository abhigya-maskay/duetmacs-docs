---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-011]
---

# ADR-011: Request Semantics & Concurrency v1

**Status**: Accepted  
**Date**: 2025-10-03

## Context
- Story 003 defines concrete behavior for request handling needed by Emacs clients and tests.
- ADR-006 chooses STM for general state management but does not constrain the daemon’s v1 scheduling.

## Options
- Single-threaded sequential dispatcher vs. concurrent worker pool.
- Support JSON-RPC batch arrays vs. reject in v1.
- Acceptable ID types and ordering guarantees.

## Decision Scope
- In scope: Concurrency model, batching, notifications, request IDs, response ordering, initialization requirements.
- Out of scope: Transport/framing (ADR-010), detailed error mapping (ADR-013).

## Decision
- Execution model: Single-threaded, sequential; one request at a time; preserve arrival order.
- Batching: Not supported; a JSON array request returns −32600 with message "Batch requests not supported".
- Notifications: Supported; process without response. Unknown-method notifications are logged at `warn` and do not terminate the daemon.
- Request IDs: Accept `string | number | null`; echo exactly. Reject `object | array | boolean` with −32600.
- Ordering: Responses are sent in the same order as requests were received.
- Initialization: No required handshake; all methods callable immediately.
- Client guidance: Clients SHOULD NOT pipeline concurrent requests to this v1 server.

## Consequences
- Simpler dispatcher and state; deterministic test behavior and ordering.
- Future concurrency will require explicit change management and documentation.

## Open Questions
None

## References
- Story 003: `03-Epics/V1-Foundation/01-Project-Bootstrap/Stories/003-json-rpc-daemon/003-json-rpc-daemon.md`
- ADR-006: State Management

