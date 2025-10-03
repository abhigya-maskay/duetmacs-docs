---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-014]
---

# ADR-014: Daemon Lifecycle & Signals

**Status**: Accepted  
**Date**: 2025-10-03

## Context
- Story 003 defines lifecycle guarantees needed by editor clients and automated tests.

## Options
- Hard exit on EOF/signals vs. graceful shutdown.
- Fixed shutdown window vs. best-effort without bound.
- Treat SIGHUP as reload vs. terminate in v1.

## Decision Scope
- In scope: Shutdown triggers, timing, behavior with in-flight work, exit codes, logging, cleanup.
- Out of scope: Transport/framing (ADR-010).

## Decision
- Triggers: stdin EOF; SIGINT; SIGTERM; SIGHUP → graceful shutdown.
- Timing: Complete shutdown within 2 seconds of trigger or after handling a `shutdown` request.
- Behavior:
  - Stop reading new frames immediately upon trigger.
  - If the current request is `shutdown`, reply success then exit.
  - If a non-shutdown request is running, attempt to finish quickly; if not done by 2 seconds, exit anyway.
  - Framing/parse errors: log, attempt an error response if possible, keep daemon alive.
- Exit codes: 0 for graceful shutdown; 1 for unrecoverable internal errors (e.g., initialization failure, fatal IO).
- Logging:
  - On EOF: log "stdin closed, shutting down gracefully" at info.
  - On signal: log received signal at info.
  - On shutdown: log summary and flush log sinks.
- Cleanup: Flush stdout and logs; close scribes before process exit. Cap log flush wait at ~500 ms inside the 2s window.
- SIGHUP: Treat like SIGTERM in v1 (no reload).
- `shutdown` as notification (no id): triggers graceful exit without a response.

## Consequences
- Predictable integration for editors and scripts; simple, testable lifecycle.

## Open Questions
None

## References
- Story 003: `03-Epics/V1-Foundation/01-Project-Bootstrap/Stories/003-json-rpc-daemon/003-json-rpc-daemon.md`

