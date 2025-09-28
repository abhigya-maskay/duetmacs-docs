---
tags: [component, cli, parser]
aliases: [CLI Parser]
---

# CLIParser

## Meta
- Scope date: 2025-01-08
- Author: Component Designer

## Purpose
- Parse command-line arguments, validate input, and route to appropriate subcommands or handlers.

## Responsibilities
- Parse global flags (--version, --help, --log-level, --no-color)
- Expose subcommands in help (version, doctor, rpc, prompt) for discoverability
- Route only the `version` subcommand in Story 001; others are out of scope
- Validate argument structure and generate usage errors

## Boundaries
- In-scope: Argument parsing, command routing, global flag extraction
- Out-of-scope: Command execution, output formatting, error message styling
- Inputs (conceptual): Raw command-line arguments array
- Outputs (conceptual): Parsed command structure, validation errors

## Collaborators & Dependencies
- Internal: HelpFormatter (to display help), ErrorHandler (for invalid input)
- External: optparse-applicative (CLI framework)
- Notes: Must handle both long and short flag formats; enable showHelpOnEmpty and showHelpOnError; global flags: --version, --help, --log-level, --no-color. Story 001 implements only `version` and global help; `doctor`, `rpc`, and `prompt` behavior is deferred. Built-in parser failures may bubble through `optparse-applicative`’s default error renderer without additional formatting.

## Risks & Open Questions
- Decision: Use optparse-applicative; rely on its built-in per-command help handling.
- Note: Subcommand help routing handled by framework; minimal custom logic needed in Story 001.
