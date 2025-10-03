---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-013]
---

# ADR-013: Daemon Lifecycle & Signals

**Status**: Accepted  
**Date**: 2025-10-03

## Context
- Story 003 defines lifecycle guarantees for editor clients and automated tests.

## Options
- Hard exit on EOF/signals vs. graceful shutdown.
- Fixed shutdown window vs. best-effort without bound.
- Treat SIGHUP as reload vs. terminate in v1.

## Decision Scope
- In scope: Shutdown triggers, timing, behavior with in-flight work, exit codes, logging, cleanup.
- Out of scope: Transport/framing ([[ADR-010 RPC IO Framing & Stdout Discipline|ADR-010]]).

## Decision
- Triggers: stdin EOF; SIGINT; SIGTERM; SIGHUP → graceful shutdown.
- Timing: 2 seconds from trigger or after handling `shutdown` request.
- Behavior:
  - Stop reading new frames immediately upon trigger.
  - If current request is `shutdown`, reply `{"message": "Shutting down gracefully"}` then exit.
  - If non-shutdown request running, attempt to finish and send response; if not done by 2 seconds, exit without response.
  - Framing/parse errors: log, attempt error response, keep daemon alive.
- Exit codes: 0 for graceful shutdown; 1 for unrecoverable internal errors (e.g., initialization failure, fatal IO).
- Logging:
  - On EOF: log "stdin closed, shutting down gracefully" at info.
  - On signal: log signal name at info.
  - On shutdown: log summary and flush sinks.
- Cleanup: Flush stdout and logs; close scribes before exit. Cap log flush at 500 ms; exit anyway if incomplete to meet 2-second deadline.
- SIGHUP: Treat like SIGTERM in v1 (no reload).
- `shutdown` notification: triggers graceful exit without response; follows same 2-second window as `shutdown` request.

## Consequences
- Predictable integration for editors and scripts; simple, testable lifecycle.

## Open Questions
None

## References
- Story 003: `03-Epics/V1-Foundation/01-Project-Bootstrap/Stories/003-json-rpc-daemon/003-json-rpc-daemon.md`

