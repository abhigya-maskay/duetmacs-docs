---
tags: [component, cli, logging]
aliases: [Logger]
---

# Logger

## Meta
- Scope date: 2025-01-08
- Author: Component Designer

## Purpose
- Provide structured logging with configurable levels and output destinations for debugging and monitoring.

## Responsibilities
- Initialize logging with appropriate default level (warn)
- Route logs to stderr or file when `DUET_RPC_LOG` is set
- Format log entries with timestamp, level, and message
- Respect the `--log-level` flag

## Boundaries
- In-scope: Log level filtering, output routing, structured formatting
- Out-of-scope: Log aggregation, remote logging
- Inputs (conceptual): Log level, message, context data
- Outputs (conceptual): Formatted log entries to configured destination

## Collaborators & Dependencies
- Internal: None (log formatting is independent of OutputFormatter in Story 001)
- External: katip (structured logging), environment variables
- Notes: Use katip; default scribe stderr; default level warn; when `DUET_RPC_LOG` is set, write to that file; guard expensive debug logs; no built-in rotation in Story 001 (rely on external rotation).

## Risks & Open Questions
- Decision: No built-in log rotation in Story 001; rely on external rotation.
- Risk: Debug logging can add overhead — mitigate by guarding construction of debug messages.
- Risk: Invalid/unwritable `DUET_RPC_LOG` path — fallback to stderr with a single clear warning, without affecting command output.
