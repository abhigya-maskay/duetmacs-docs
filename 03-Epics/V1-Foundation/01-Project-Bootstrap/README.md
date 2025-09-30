---
tags: [epic, v1-foundation, bootstrap, project-setup]
aliases: [Project Bootstrap Epic, Bootstrap Epic, Epic 1, 01-Project Bootstrap]
---

# Epic: Project Bootstrap (CLI + Emacs Plugin)

## Overview
Establish working skeletons for duet-rpc (our CLI) and the Emacs package with tooling, packaging, CI, and a minimal RPC handshake so both can run end-to-end in a no-op mode.

## Goal
Create the foundational infrastructure for both the CLI tool and Emacs integration, establishing the basic communication channel and core utilities that all other features will build upon.

## Stories Within This Epic

### Development Stories
1. **[[Stories/001-cli-version-help-bootstrap/001-cli-version-help-bootstrap|Story 001: CLI Version/Help Bootstrap]]** – Establishes the minimal CLI with version/help commands, shared logging, and output formatting infrastructure.

### Components Defined
- Output Formatter
- Version Manager
- Logger
- Config Skeleton
- Help Formatter
- CLI Parser

## Scope

### CLI Scope (duet-rpc)
- Binary executable: `duet-rpc`
- Language toolchain setup
- Subcommands: `version` command and global `--version`/`--help`
- Logging infrastructure with configurable destinations
- Color-aware output formatter honoring TTY detection and opt-out flags
- Config path constants documented for the loader skeleton

## Dependencies
None - this is the foundational epic.

## Acceptance Criteria

### CLI
- `duet-rpc --version` prints the semantic version and exits 0
- `duet-rpc version` mirrors the same output
- `duet-rpc --help` shows synopsis and available subcommands (`version`, `doctor`, `rpc`, `prompt` placeholders) routed through the shared formatter
- No-argument invocation prints help and exits 0
- Unknown subcommands surface optparse-applicative usage errors without stack traces
- Output honors `NO_COLOR`, `--no-color`, and TTY detection
- Logging defaults to warn level on stderr and supports `--log-level` overrides and `DUET_RPC_LOG`

## Phase
V1-Foundation

## Status
Planning (90% specified)

## Related Documentation
- [[Epic Roadmap]] - Project roadmap overview
- [[System Overview]] - Architecture context

---
*Navigation: [[00-Start-Here/README|Home]] > [[03-Epics]] > [[03-Epics/V1-Foundation/README|V1-Foundation]] > Project Bootstrap*
