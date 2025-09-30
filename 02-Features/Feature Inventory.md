---
tags: [features, capabilities, inventory]
aliases: [Feature List, Capabilities]
---

# Feature Inventory — Project Bootstrap Scope

The current focus is Story 001, which delivers a minimal yet reliable CLI foundation. Features below represent the live and planned capabilities within this epic.

## Delivered (Story 001 - Completed 2025-09-29)
- `duet-rpc --version` flag and `version` command share the semantic version sourced from Cabal metadata.
- Global `--help` output with command synopsis, help footer, and color-aware formatting.
- Shared OutputFormatter honors TTY detection, `NO_COLOR`, and `--no-color` overrides.
- Logging subsystem defaults to warn level on stderr and accepts `--log-level` overrides.
- `DUET_RPC_LOG` environment variable redirects logs with graceful fallback on failure.

## Components Status (As of Story 001 Completion)

### Implemented
- **Output Formatter**: Terminal output with color management and TTY detection
- **Version Manager**: Version information from package manifest
- **Logger**: Structured logging with level control
- **Help Formatter**: Help text generation and formatting
- **CLI Parser**: Command-line argument parsing and routing

### In Progress
- **Config Loader**: Skeleton ready - config path constants and defaults defined, no file I/O yet
- **Error Handler**: Basic error formatting in progress

## Pending Implementation
- Full config loading with file I/O (beyond Story 001)
- Complete error handling with all error types
- RPC components (RPC Handler, Session Manager, etc.)
- Emacs integration components
