---
tags: [features, capabilities, inventory]
aliases: [Feature List, Capabilities]
---

# Feature Inventory — Project Bootstrap Scope

The current focus is Story 001, which delivers a minimal yet reliable CLI foundation.

## Story 001: CLI Version/Help Bootstrap (Completed)

### Core CLI Features Delivered
- `duet-rpc --version` flag and `version` command outputting semantic version from Cabal metadata
- `duet-rpc --help` showing command synopsis with subcommands (version, doctor, rpc, prompt)
- Default behavior: no arguments shows help and exits 0
- Unknown command/flag handling: error message + help, exits 1, no stack traces

### Infrastructure Components Delivered
- **Logging Infrastructure**:
  - Structured logging to stderr (warn level default)
  - `--log-level` flag support (debug|info|warn|error)
  - `DUET_RPC_LOG` environment variable for file output
  - Graceful fallback to stderr when log file unavailable

- **Output Formatting**:
  - TTY/pipe detection at startup
  - `NO_COLOR` environment variable support
  - `--no-color` global flag
  - Color palette: success (green), warning (yellow), error (red), info (cyan), dimmed (gray)
  - Unicode fallback symbols: OK→[OK], WARN→[WARN], FAIL→[FAIL], INFO→[INFO]

- **Config Skeleton**:
  - Config search paths defined (constants only, no loading)
  - Config struct with defaults created
  - Precedence documented: DUET_RPC_CONFIG → project .duet-rpc.toml → XDG/home config

## Components Implemented (Story 001)

### Fully Implemented
- **CLI Parser**: Command-line argument parsing with optparse-applicative
- **Output Formatter**: Terminal output with color management and TTY detection
- **Version Manager**: Version information from Cabal package manifest
- **Logger**: Structured logging with level control and file output
- **Help Formatter**: Help text generation with color-aware formatting

### Partially Implemented (Skeleton Only)
- **Config Loader**: Path constants and defaults defined, no file I/O yet
- **Error Handler**: Basic error message formatting without stack traces

## Beyond Story 001 Scope

### Not Yet Started
- Doctor command implementation
- RPC command implementation
- Prompt command implementation
- Full configuration loading with file I/O
- RPC components (Handler, Session Manager, Protocol Framer)
- Emacs integration components
- CI/CD setup and packaging
