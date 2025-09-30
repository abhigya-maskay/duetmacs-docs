---
tags: [overview, introduction, architecture]
aliases: [Overview, About, Introduction]
---

# Project Overview

## What is DuetMacs?

DuetMacs is an AI-assisted development environment that integrates advanced language models directly into Emacs. It provides intelligent code generation, refactoring, and analysis capabilities through a robust RPC architecture.

## Core Components (Story 001 Scope)

### 1. CLI Bootstrap (duet-rpc)
The CLI binary provides:
- `--version` flag and `version` command sourced from Cabal metadata
- Global `--help` output with synopsis, command listing, and footer
- Logging controls (`--log-level`, `DUET_RPC_LOG`) with structured Katip output
- Output formatting that respects TTY detection, `NO_COLOR`, and `--no-color`
- Config skeleton documenting search paths and precedence (no file I/O yet)

### 2. Shared Infrastructure
- OutputFormatter and HelpFormatter establish consistent terminal rendering
- VersionManager, ErrorHandler, and CLIParser compose the command registry
- ConfigLoader skeleton defines types/defaults while remaining pure

### 3. Emacs Integration (Documented)
The Emacs UI architecture remains documented to guide upcoming work, but implementation resumes after the CLI bootstrap milestone.

## Current Capabilities
- Build and run `duet-rpc` CLI with version/help surfaces
- Structured logging with stderr default and file redirection fallback
- Color-aware output formatting honouring opt-out flags and pipes
- Config precedence constants for upcoming loader expansion
- Risk, testing, and UX notes scoped strictly to Story 001

## Work in Flight
- Formalizing config loader behaviours beyond constants and defaults
- Finalizing CI coverage for version/help, logging, and formatter scenarios
- Maintaining alignment between documentation, ADRs, and CLI implementation

## Design Principles

1. **Safety First**: All operations are reversible with automatic backups
2. **Predictable**: Clear configuration precedence and deterministic behavior
3. **Performant**: Sub-second response times for common operations
4. **Extensible**: Hook system for customization without forking
5. **Integrated**: Deep Emacs integration, not a bolt-on solution

## Technology Stack

- **Frontend**: Emacs Lisp with transient menus
- **Backend**: Haskell with GHC 9.10
- **Protocol**: JSON-RPC 2.0 over stdio
- **Build**: Cabal with reproducible builds
- **Testing**: Property-based testing with Hedgehog

## Project Status

Current Phase: **V1-Foundation — CLI Bootstrap**
- 🚀 Story 001 (CLI version/help) delivered 2025-09-29
- 📝 Supporting documentation, UX notes, testing strategy, and risk register maintained alongside the CLI
- 🧪 Additional tests and CI automation under development

## Related Documentation

- [[System Overview]] - Technical architecture
- [[Epic Roadmap]] - Development timeline
- [[Feature Inventory]] - Complete capability list

## Terminology

- **duet-rpc**: Our RPC/CLI binary
- **codex**: OpenAI's CLI tool
- **claude-code**: Anthropic's CLI tool
- **Epic**: A vertical slice delivering end-to-end functionality
- **Session**: A persistent conversation with the AI

---
*Navigation: [[00-Start-Here/README|Home]] > Project Overview*
