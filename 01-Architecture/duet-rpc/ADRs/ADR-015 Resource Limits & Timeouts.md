---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-015]
---

# ADR-015: Resource Limits & Timeouts

**Status**: Accepted  
**Date**: 2025-10-03

## Context
- Story 003 establishes request size and read timing limits against abusive or buggy clients.

## Options
- Fixed vs. configurable limits in v1.
- Separate header-size cap vs. total message cap only.
- Buffer across timeouts vs. discard partial frames.

## Decision Scope
- In scope: Request size cap, header-size cap, read timeout, behavior when limits are exceeded, logging.
- Out of scope: Concurrency and retries ([[ADR-011 Request Semantics & Concurrency v1|ADR-011]]).

## Decision ([[ADR-010 RPC IO Framing & Stdout Discipline|ADR-010]] for framing behavior, [[ADR-014 Logging & Diagnostics Policy|ADR-014]] for redaction)
- Max request size: 10 MB. Content-Length > 10 MB → respond −32600 with `id: null` without reading body; discard payload and continue.
- Read timeout: 30 seconds per framed message. On timeout: discard partial data, log warning, continue reading (connection remains open; multiple timeouts do not terminate daemon).
- Header cap: 8 KB for header section; exceed → respond −32600 with `id: null`, discard, continue.
- Partial frames: never buffer across timeouts; reset framing state after logging.
- Logging: log reason and observed sizes/timing; do not include payload bytes.
- Configuration: Values fixed in v1 (no env/config knobs).

## Consequences
- Resilient IO loop with predictable behavior under stress and error conditions.
- Clear test matrix for oversize, timeout, header abuse, and recovery.

## Open Questions
None

## References
- Story 003: `03-Epics/V1-Foundation/01-Project-Bootstrap/Stories/003-json-rpc-daemon/003-json-rpc-daemon.md`
- [[ADR-010 RPC IO Framing & Stdout Discipline|ADR-010]]: RPC I/O Framing & Stdout Discipline

