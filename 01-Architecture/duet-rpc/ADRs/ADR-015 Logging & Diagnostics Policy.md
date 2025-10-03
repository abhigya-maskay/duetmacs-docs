---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-015]
---

# ADR-015: Logging & Diagnostics Policy

**Status**: Accepted  
**Date**: 2025-10-03

## Context
- We need structured, configurable logging that never contaminates stdout, and runtime control of verbosity as per Story 003.
- ADR-004 selected Katip as the logging library.

## Options
- Logger sinks: stderr vs. file, colored vs. plain.
- Level model and normalization; runtime changes via RPC.
- Log field set: minimal vs. verbose.

## Decision Scope
- In scope: Logger choice, sinks, level names, runtime control, formatting, fields, redaction, startup logging.
- Out of scope: Metrics/tracing stack.

## Decision
- Logger: Katip.
- Sinks: default to stderr; if `DUET_RPC_LOG` points to a writable file, log there; on failure, warn and fall back to stderr. Never write logs to stdout.
- Levels: supported names `debug|info|warn|error` (case-insensitive); normalized to lowercase; map to Katip severities accordingly.
- Dynamic changes: RPC `setLogLevel` adjusts the active level immediately for the process lifetime; sinks are not changed at runtime.
- Formatting & color: stderr uses human-readable format with color only when TTY and color enabled; honor `--no-color` and `NO_COLOR`. File sink uses plain text (no color codes).
- Fields (minimum): `timestamp`, `severity`, `method` when known, `id` (including null), `message`.
- Severity guidance: client-caused errors (parse/invalid-request/invalid-params/unknown-method) → warn; internal faults → error; lifecycle events → info.
- Privacy/redaction: do not log payload bodies on parse/oversize failures; never log secrets, tokens, or absolute user paths.
- Startup: on boot, log version, PID, active level, and chosen sink (stderr | file path).

## Consequences
- Predictable diagnostics that respect stdout discipline and user preferences.

## Open Questions
None

## References
- Story 003: `03-Epics/V1-Foundation/01-Project-Bootstrap/Stories/003-json-rpc-daemon/003-json-rpc-daemon.md`
- ADR-004: Core Libraries

