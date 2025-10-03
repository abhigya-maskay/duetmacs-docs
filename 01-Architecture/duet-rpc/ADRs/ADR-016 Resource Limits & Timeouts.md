---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-016]
---

# ADR-016: Resource Limits & Timeouts

**Status**: Accepted  
**Date**: 2025-10-03

## Context
- Story 003 establishes concrete limits for request size and read timing to harden the daemon against abusive or buggy clients.

## Options
- Fixed vs. configurable limits in v1.
- Separate header-size cap vs. total message cap only.
- Buffer across timeouts vs. discard partial frames.

## Decision Scope
- In scope: Request size cap, header-size cap, read timeout, behavior when limits are exceeded, logging.
- Out of scope: Concurrency and retries (ADR-011).

## Decision
- Max request size: 10 MB. If exceeded, respond with −32600, discard payload, and continue running.
- Read timeout: 30 seconds per framed message (headers + body). On timeout, discard partial data, log a warning, and continue.
- Header cap: 8 KB maximum for the header section; exceed → −32600 and discard.
- Partial frames: never buffer across timeouts; always reset framing state after logging.
- Logging: log reason and observed sizes/timing; do not include payload bytes.
- Configuration: Values are fixed in v1 (no env/config knobs); revisit in a future version if needed.

## Consequences
- Resilient IO loop with predictable behavior under stress and error conditions.
- Clear test matrix for oversize, timeout, header abuse, and recovery.

## Open Questions
None

## References
- Story 003: `03-Epics/V1-Foundation/01-Project-Bootstrap/Stories/003-json-rpc-daemon/003-json-rpc-daemon.md`
- ADR-010: RPC I/O Framing & Stdout Discipline

