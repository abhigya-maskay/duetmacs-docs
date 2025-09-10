---
tags: [overview, introduction, architecture]
aliases: [Overview, About, Introduction]
---

# Project Overview

## What is DuetMacs?

DuetMacs is an AI-assisted development environment that integrates advanced language models directly into Emacs. It provides intelligent code generation, refactoring, and analysis capabilities through a robust RPC architecture.

## Core Components

### 1. Emacs UI (ELisp)
The frontend that provides:
- Editor integration and keybindings
- Command palette for discoverability
- Context capture (region/buffer/file/project)
- Diff review and approval workflows
- Real-time status and progress display

### 2. RPC CLI Core (duet-rpc)
A Haskell-based backend that handles:
- Session orchestration and state management
- Prompt assembly and context building
- AI provider integration (OpenAI, Anthropic)
- Token tracking and rate limiting
- Safe patch generation with backups

### 3. Optional Adapters (Future)
Planned extensions:
- MCP adapter for cross-client integration
- Local daemon for shared caching
- HTTP/gRPC endpoints for remote access

## Key Features

### Chat & Sessions
- Interactive coding sessions with AI
- Persistent conversation history
- Session branching and organization

### Context Management
- Smart context ingestion from files/projects
- Respect for .gitignore and custom excludes
- Template-based prompt construction

### Safety & Controls
- Dry-run mode by default
- Write protection with allowlists
- Automatic backups before changes
- Size and file count limits

### Multi-Model Support
- Provider switching (OpenAI, Anthropic)
- Per-project model configuration
- Token budget management

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

Currently in **Planning** phase:
- 📝 Architecture and design documentation
- 📝 Epic and story specifications
- 📝 Component identification
- 📝 Technology stack selection

## Related Documentation

- [[System Overview]] - Technical architecture
- [[Feature Development Workflow]] - How we build features
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
