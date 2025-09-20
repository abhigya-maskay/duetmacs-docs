---
tags: [component, cli, error]
aliases: [Error Handler]
---

# ErrorHandler

## Meta
- Scope date: 2025-01-08
- Author: Component Designer

## Purpose
- Provide consistent error formatting across all CLI operations.

## Responsibilities
- Format error messages for user-friendly display
- Suppress stack traces in production mode
- Display usage hints on command errors
- Note: Does not manage external subprocess lifecycle (spawn/retry handled by Emacs)

## Boundaries
- In-scope: Error formatting, usage hint generation
- Out-of-scope: Managing external subprocess lifecycle (spawn/retry), detailed system diagnostics
- Inputs (conceptual): Error type, context, command state
- Outputs (conceptual): Formatted error message

## Collaborators & Dependencies
- Internal: OutputFormatter (for error styling), HelpFormatter (for usage hints)
- External: Process exit APIs
- Notes: Ensures consistent error experience; no stack traces for user errors; for usage errors print parser diagnostics and "Use --help" hint; in debug log-level include exception details.

## Risks & Open Questions
- Decision: Exit codes 0/2/1 mapping confirmed for Story 001.
- Decision: No stack traces in normal mode; include details only in debug mode via logs.
- Decision: English-only error messages in Story 001; i18n later.
