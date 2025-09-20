---
tags: [operations, error-handling, planned]
aliases: [Error Handling, Error Management]
---

# Error Handling

**Status**: Planned (Design Phase)
Scope: Story 001 only (CLI Version/Help Bootstrap)

## Overview
Error handling for Story 001 focuses solely on the minimal CLI surface: `--version`, `version`, and `--help` (help shown by default with no args). The goal is clear user feedback and no stack traces.

## Scope (Story 001)

### Error Categories (Story 001)
- **Usage Errors** — Unknown subcommand/flag, invalid arguments. Show a concise error followed by usage/help.
- **Log Initialization Warning** — Invalid/unwritable `DUET_RPC_LOG` path. Warn once, fallback to stderr.
- **Internal Errors (unexpected)** — Treat as a generic runtime error without stack trace.

### Error Handling Strategy
- Prefer graceful handling; never print stack traces to users
- Usage errors show a brief message and then help
- Log initialization failures do not fail the command; fallback to stderr with a single warning
- Keep messages concise and action-oriented

### Key Requirements (Story 001)
- Exit codes:
  - `0` — Success
  - `2` — Usage error (unknown command/flag, invalid args)
  - `1` — Generic runtime error (unexpected)
- No stack traces on user-facing output
- No sensitive data in error messages

### Out of Scope (removed for Story 001)
- Provider/network errors
- Timeouts and RPC-related codes
- File I/O categories beyond log initialization fallback

### Implementation Notes (Story 001)
- [[05-Components/CLI-Components/06-error-handler|Error Handler Component]] — surface concise errors and show help for usage errors
- Use CLI framework defaults where suitable (usage/help rendering)
- Log warnings/errors via logger; do not leak sensitive details

## Related Documentation
- [[08-Operations/README|Operations Overview]]
- [[08-Operations/Observability|Observability]] — Logging behavior for Story 001
- [[07-Testing/Test Strategy|Test Strategy]] — Usage-error and logging fallback tests

---
*Navigation: [[00-Start-Here/README|Home]] > [[08-Operations]] > Error Handling*
