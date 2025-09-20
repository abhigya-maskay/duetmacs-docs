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
1. **[[Stories/001-cli-version-help-bootstrap/001-cli-version-help-bootstrap|Story 001: CLI Version/Help Bootstrap]]** - Basic CLI with version/help commands
2. **[[Stories/002-emacs-init-health/002-emacs-init-health|Story 002: Emacs Init Health]]** - Emacs package initialization and health checks  
3. **[[Stories/003-cli-doctor/003-cli-doctor|Story 003: CLI Doctor]]** - Environment verification command
4. **[[Stories/004-cli-rpc-ping/004-cli-rpc-ping|Story 004: CLI RPC Ping]]** - Basic RPC connectivity test
5. **[[Stories/005-emacs-subprocess-mgmt/005-emacs-subprocess-mgmt|Story 005: Emacs Subprocess Management]]** - Process lifecycle handling
6. **[[Stories/006-cli-prompt-dry-run/006-cli-prompt-dry-run|Story 006: CLI Prompt Dry Run]]** - No-op prompt processing
7. **[[Stories/007-emacs-command-palette/007-emacs-command-palette|Story 007: Emacs Command Palette]]** - Command discovery interface
8. **[[Stories/008-ci-smoke-checks/008-ci-smoke-checks|Story 008: CI Smoke Checks]]** - Continuous integration setup

### Components Defined
- Output Formatter
- Version Manager
- Logger
- Config Loader
- Help Formatter
- Error Handler
- CLI Parser

## Scope

### UI Scope (Emacs)
- Package scaffolding (metadata, init, keymap)
- Command palette entry
- Subprocess management to launch CLI
- Version/health commands
- Basic minibuffer status
- `*DUET RPC*` control buffer (shows status/PID)
- `*DUET Logs*` append-only log buffer
- Transient-style `duet-dispatch` menu
- Install/build instructions

### CLI Scope (duet-rpc)
- Binary executable: `duet-rpc`
- Language toolchain setup
- Subcommands: `version`, `doctor`, `rpc --ping`, `prompt --dry-run`
- Logging infrastructure
- Config loading precedence
- Packaging/release scripts

## Dependencies
None - this is the foundational epic that all others depend on.

## Acceptance Criteria

### CLI
- `duet-rpc --version` prints version
- `duet-rpc doctor` checks environment and prints checklist
- Emits stable JSON with `--json` flag
- Respects `NO_COLOR`/non-TTY environments
- `duet-rpc rpc --ping` responds with pong
- `duet-rpc prompt --file <f> --dry-run` returns no-op response
- Help behavior follows Unix conventions

### Emacs
- Package loads successfully
- `M-x duet-rpc-version` shows duet-rpc version
- Commands to start/stop duet-rpc subprocess
- Health check shows connected/ping OK
- Basic command palette entry present

### CI
- Lints/builds/tests run on PR
- `duet-rpc doctor` included in smoke checks
- Release artifacts produced for CLI
- Packaging instructions validated

## Phase
V1-Foundation

## Status
Planning (90% specified)

## Related Documentation
- [[Epic Roadmap]] - All epics overview
- [[Feature Development Workflow]] - Development process
- [[System Overview]] - Architecture context

---
*Navigation: [[00-Start-Here/README|Home]] > [[04-Epics]] > [[04-Epics/V1-Foundation/README|V1-Foundation]] > Project Bootstrap*
