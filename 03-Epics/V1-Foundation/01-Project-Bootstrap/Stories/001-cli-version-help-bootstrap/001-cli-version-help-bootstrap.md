---
tags: [story, cli, bootstrap, foundation]
aliases: [Story 001, CLI Bootstrap, Version Help Story]
---
# Story 001: CLI Version/Help Bootstrap

As a CLI user, I want a minimal `duet-rpc` binary that builds and exposes version and help so that I can verify installation and discover available commands.

## Priority
- Must

## Dependencies
- None

## Non-Goals
- Implementing behavior for doctor, rpc, or prompt commands
- Full configuration loading (just skeleton/structure for [[Config Loader]])
- Packaging/release or Emacs integration

## Business Rationale
- Ensures the toolchain is correctly installed and discoverable
- Establishes a testable CLI entrypoint to anchor ongoing CLI work
- Provides foundational services (logging, output formatting) for all commands

## Additional Scope (Foundation Elements)

### Logging Infrastructure
- Initialize logging to stderr (warn level default)
- Support `--log-level` (debug|info|warn|error) global flag
- Support `DUET_RPC_LOG` env var for file output
- Structured log format with timestamp, level, and message
- Ongoing CLI work uses this [[04-Components/CLI-Components/03-logger|logger component]]

### Output Formatting
- Detect TTY/pipe at startup
- Honor NO_COLOR env var and --no-color global flag
- Initialize shared [[04-Components/CLI-Components/01-output-formatter|formatter]] for all commands
- Define color palette: success (green), warning (yellow), error (red), info (cyan), dimmed (gray)
- Fallback symbols when no unicode: OK→[OK], WARN→[WARN], FAIL→[FAIL], INFO→[INFO]
- Use the `doctor` command snapshot as the representative OutputFormatter output to validate colorized vs plain rendering until a direct `--help` snapshot exists (the placeholder implementation is sufficient).

### Config Skeleton
- Define config search paths (not loading yet, just constants)
- Create config struct with defaults
- Document precedence: DUET_RPC_CONFIG env → project .duet-rpc.toml → XDG/home config
- Prepare structure for validation tooling
- Foundation for [[04-Components/CLI-Components/04-config-loader|configuration component]]

## Acceptance Criteria (Given/When/Then)
- Given the binary is installed, When I run `duet-rpc --version`, Then it prints a semantic version (e.g., `0.1.0`) and exits with code 0.
- Given the binary is installed, When I run `duet-rpc version`, Then it prints the same semantic version and exits with code 0.
- Given the binary is installed, When I run `duet-rpc --help`, Then it shows a synopsis and lists subcommands `version`, `doctor`, `rpc`, and `prompt` with one-line descriptions, and exits 0.
- Given no arguments, When I run `duet-rpc`, Then it prints the help text and exits 0.
- Given the help output, When I read it, Then it clearly indicates that `doctor`, `rpc`, and `prompt` are available subcommands (behaviors outside Story 001 remain placeholders).
- Given an unknown subcommand or flag is provided, When I run `duet-rpc <unknown>`, Then the CLI prints an error followed by usage/help, exits with status `1` (optparse-applicative's usage-error default), and does not print a stack trace.
- Given any output is printed, Then it is newline-terminated and routed through the shared ShellFormatter; help output follows the same color policy as other commands (colorized on a TTY, honoring `NO_COLOR`/`--no-color`, and plain when piped), with coverage exercised via the `doctor` command snapshot.
 - Given a help synopsis is shown, Then it includes `duet-rpc [COMMAND] [OPTIONS]` and a footer: `See 'duet-rpc <command> --help' for more information.`
 - Given typical developer hardware, Then the above commands respond within ~100ms.
 - Given `--log-level debug`, When I run `duet-rpc --log-level debug version`, Then structured debug logs appear on stderr with timestamp, level, and message.
 - Given no log flags, When I run any command, Then only warnings and errors are logged to stderr by default.
- Given `DUET_RPC_LOG=/tmp/duet.log`, When I run any command, Then logs are written to the specified file instead of stderr.
- Given `DUET_RPC_LOG` points to an invalid or unwritable path, When I run any command, Then logging falls back to stderr with a clear single warning about the log file failure (no stack trace) and the command output remains unchanged.
 - Given `NO_COLOR=1` or piped output, When I run `duet-rpc --help`, Then output contains no ANSI color codes.
 - Given a TTY without NO_COLOR, When I run `duet-rpc --help`, Then output includes colors for headings and commands.
 - Given `--no-color` flag, When I run `duet-rpc --no-color --help`, Then output contains no ANSI codes regardless of TTY.

## Assumptions / Open Questions
- Assumption: Project uses a single source of truth for version (e.g., `--version` reads from build metadata) — Confidence: high — Impact if wrong: duplicate version sources drift — Validation: choose and document a single version authority.
- Decision: CLI framework is optparse-applicative (Haskell).
- Decision: Version authority is the Cabal package version (via the generated Paths module); help layout uses optparse-applicative defaults.

## Defaults
- No args shows help and exits 0.
- `-h/--help` and `-V/--version` supported globally.

## Success Metrics
- `duet-rpc --version` and `duet-rpc --help` run successfully in CI on all target platforms.
- Developer can build and run the binary locally with the same outputs.
