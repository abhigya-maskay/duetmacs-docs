---
tags: [ux, story/001, cli, bootstrap]
aliases: [Story 001 UX, CLI Bootstrap UX, Version Help UX]
---

# Story 001: CLI Version/Help Bootstrap
UX need: Light

Phase: Verify install and discover commands, Goal: Confirm binary responds and exposes version/help
Entry: User has terminal access and `duet-rpc` installed; Exit: Version printed; help lists subcommands.

Steps
- User runs `duet-rpc --version` → prints `0.1.0`
- User runs `duet-rpc version` → prints `0.1.0`
- User runs `duet-rpc --help` or `duet-rpc` → prints help

Touchpoints
- Commands: `duet-rpc --version`, `duet-rpc version`, `duet-rpc --help`, `duet-rpc`
- Global options: `-V/--version`, `-h/--help`
- Global flags: `--log-level` (debug|info|warn|error), `--no-color`
- Environment variables: `DUET_RPC_LOG` (file path), `NO_COLOR` (1 to disable colors)

Copy
- Version output: `0.1.0`
- Help synopsis: `duet-rpc [COMMAND] [OPTIONS]`
- version: Print version information
- doctor: Diagnose environment (not yet implemented)
- rpc: Start RPC server (not yet implemented)
- prompt: Run prompt tools (not yet implemented)
- Footer: See 'duet-rpc <command> --help' for more information.

Patterns and States
- Success: outputs match above
- Error: unknown subcommand/flag → error + usage; no stack traces
- Color: colorized help when TTY; honor `NO_COLOR`; plain when piped
- Logging: Default warn level to stderr; `--log-level` overrides; `DUET_RPC_LOG` redirects to file
- Log format: Structured with timestamp, level, message (e.g., `2024-01-15T10:30:45Z WARN Command not found`)

Alt/Recovery
- Invalid input: show error + usage; suggest `--help`
- No args: show help
- Debug issues: Use `--log-level debug` for verbose logging
- Log to file: Set `DUET_RPC_LOG=/path/to/file.log`

Data/Decisions
- Version authority: package manifest (e.g., Cargo.toml)
- Help layout: CLI framework defaults (e.g., Clap)

Defaults
- No args → help
- `-h/--help`, `-V/--version`, `--log-level`, `--no-color` supported globally
- Default log level: warn (only warnings/errors shown)
- Color output: Auto-detect TTY; disabled by `NO_COLOR=1` or `--no-color`

Integration
- None; telemetry not collected for this story

GSM
- Goal: Provide discoverable CLI entrypoint
- Signals: Correct text across platforms
- Metrics: CI checks for `--version`/`--help` pass; local smoke test documented
