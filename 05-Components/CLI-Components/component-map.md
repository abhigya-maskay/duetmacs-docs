# Component Map

## Overview
- Feature/Context: CLI Version/Help Bootstrap (Story 001)
- Scope date: 2025-01-08
- Author: Component Designer

## Components
- CLIParser: Purpose — Parse command-line arguments; expose subcommands for help; route `version` only in Story 001. Boundary — Application entry. Key deps — HelpFormatter, ErrorHandler.
- VersionManager: Purpose — Manage version information from package manifest. Boundary — Domain version handling. Key deps — None (reads manifest).
- HelpFormatter: Purpose — Generate and format help text for commands. Boundary — Presentation formatting. Key deps — OutputFormatter.
- OutputFormatter: Purpose — Handle terminal output with color and symbol management. Boundary — Infrastructure I/O. Key deps — None (system calls).
- Logger: Purpose — Provide structured logging with level control. Boundary — Infrastructure logging. Key deps — None.
- ConfigLoader: Purpose — Define config paths/types and defaults (skeleton only). Boundary — Infrastructure config (no I/O in Story 001). Key deps — None.
- ErrorHandler: Purpose — Format errors consistently. Boundary — Application error handling. Key deps — OutputFormatter.

## Notes
- Decisions:
  - CLI framework: optparse-applicative with showHelpOnEmpty/showHelpOnError.
  - Version: embed via Cabal Paths (package version only in Story 001).
  - Output: ansi-terminal; cache TTY detection; honor NO_COLOR/--no-color; DUET_RPC_ASCII override.
  - Logging: katip; default scribe stderr; file scribe when `DUET_RPC_LOG` is set; default level warn; no built-in rotation in Story 001.
  - Config: Document precedence (flags > env > project `.duet-rpc.toml` > user `~/.config/duet-rpc/config.toml` > defaults); actual parsing/validation deferred to Story 003.
- Assumptions: CLI framework provides base help structure.
