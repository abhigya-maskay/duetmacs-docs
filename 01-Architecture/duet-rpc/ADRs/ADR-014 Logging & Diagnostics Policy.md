---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-014]
---

# ADR-014: Logging & Diagnostics Policy

**Status**: Accepted  
**Date**: 2025-10-03

## Context
- Structured, configurable logging that never contaminates stdout with runtime verbosity control.
- [[ADR-004 Core Libraries|ADR-004]] selected Katip as logging library.

## Options
- Logger sinks: stderr vs. file, colored vs. plain.
- Level model and normalization; runtime changes via RPC.
- Log field set: minimal vs. verbose.

## Decision Scope
- In scope: Logger choice, sinks, level names, runtime control, formatting, fields, redaction, startup logging.
- Out of scope: Metrics/tracing stack.

## Decision
- Logger: Katip.
- Sinks: default stderr; if `DUET_RPC_LOG` points to writable file, log there (checked at startup); on failure, warn to stderr and fall back to stderr. Never write logs to stdout.
- Levels: supported names `debug|info|warn|error` (case-insensitive); normalized to lowercase.
- Dynamic changes: RPC `setLogLevel` adjusts active level immediately; returns `{"level": "debug", "success": true}`. Sinks not changed at runtime.
- Invalid log levels: return error −32602 with `error.data` showing accepted values.
- Formatting & color: stderr uses human-readable format with color only when TTY and color enabled; honor `--no-color` and `NO_COLOR`. File sink uses plain text.
- Fields (minimum): `timestamp`, `severity`, `method` when known, `id` when parseable (including null; omit if unparseable), `message`.
- Severity guidance ([[ADR-011 Request Semantics & Concurrency v1|ADR-011]], [[ADR-012 Error Translation & Redaction|ADR-012]]): client-caused errors (parse/invalid-request/invalid-params/unknown-method) → warn; internal faults → error; lifecycle events → info.
- Privacy/redaction: do not log payload bodies on parse/oversize failures; never log secrets, tokens, or user paths.
- Startup: log version, PID, active level, and sink (stderr | file path).

## Consequences
- Predictable diagnostics that respect stdout discipline and user preferences.

## Open Questions
None

## References
- Story 003: `03-Epics/V1-Foundation/01-Project-Bootstrap/Stories/003-json-rpc-daemon/003-json-rpc-daemon.md`
- [[ADR-004 Core Libraries|ADR-004]]: Core Libraries

