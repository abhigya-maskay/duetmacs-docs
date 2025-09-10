---
tags: [reference, api, json-rpc, cli, planned]
aliases: [API Documentation, API Reference]
---

# API Documentation

**Status**: Planned (Design Phase)
Scope: Story 001 only (CLI Version/Help Bootstrap)

## Overview
Story 001 covers the minimal CLI interface for `duet-rpc`: global flags and the `version` command. JSON-RPC and other commands are out of scope.

## Scope (Story 001)

### CLI API (Story 001)
Command-line interface for `duet-rpc` — bootstrap only.

#### Global Options
- `--version, -V` — Show version information
- `--help, -h` — Show help message
- `--log-level` — Set logging level (debug|info|warn|error)
- `--no-color` — Disable colored output

#### Commands
- `version` — Display version information

### Out of Scope (removed for Story 001)
- JSON-RPC API (methods, schemas, notifications)
- `doctor`, `rpc`, `prompt` commands and options

## Related Documentation
- [[02-Architecture/duet-rpc/duet-rpc Technical Architecture|Technical Architecture]] - System design
- [[05-Components/CLI-Components/07-cli-parser|CLI Parser Component]] - CLI implementation

---
*Navigation: [[00-Start-Here/README|Home]] > [[09-Reference]] > API Documentation*
