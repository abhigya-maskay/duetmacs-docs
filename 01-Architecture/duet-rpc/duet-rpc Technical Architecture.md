---
tags: [architecture, technical, duet-rpc, backend]
aliases: [duet-rpc Architecture, Technical Architecture, Backend Architecture]
---

# Technical Architecture: duet-rpc

## Overview

`duet-rpc` is a Haskell-based RPC tool serving as the backend for AI-assisted development in Emacs. It operates in a hybrid model with both daemon mode for Emacs integration and one-shot CLI commands for scripting.

## System Architecture

```mermaid
flowchart LR
  E[Emacs] -->|JSON-RPC 2.0/stdio| D[duet-rpc daemon]
  C[CLI User] -->|commands| D
  D --> S[Session State/STM]
  D --> AI[AI Providers]
  D --> FS[File System]
  
  subgraph "Core Components"
    D --> P[Parser/CLI]
    D --> R[RPC Handler]
    D --> CTX[Context Builder]
  end
  
  subgraph "Observability"
    D --> L[Logs/katip]
  end
  
  AI -->|streaming| D
  D -->|notifications| E
```

## Key Architecture Decisions

### Process Model
- Hybrid architecture: Daemon for Emacs + one-shot CLI commands (scripting)
- Concurrency: Green threads with STM for safe state management
- Request Handling: Sequential processing (one request at a time); no batching in v1

### Communication
- Protocol: JSON-RPC 2.0 over stdio with LSP-style framing (Content-Length header, UTF-8 only)
- Limits: 10 MB request size, 8 KB headers, 30s read timeout
- I/O: Stdout for JSON-RPC only; logs to stderr or DUET_RPC_LOG
- Streaming: Progress notifications for AI token streaming

### Lifecycle
- Triggers: stdin EOF, SIGINT, SIGTERM, SIGHUP → graceful shutdown (2s window)
- Exit codes: 0 (graceful), 1 (fatal errors)
- Cleanup: Flushes logs/stdout (500ms cap) before exit

### Technology Stack
- Language: Haskell with GHC 9.12
- Build: Cabal with freeze files for reproducibility
- Core Libraries:
  - CLI: optparse-applicative
  - JSON: aeson + deriving-aeson
  - Logging: katip (structured)
  - HTTP: req (for AI providers)
  - Concurrency: async + STM
  - Config: tomland (TOML)

### Data & State
- Runtime state: STM with TVar/TMVar for concurrent safety
- Persistence: JSON files in the project directory
- Configuration: TOML with validation phase
- Backups: Timestamped .bak files before modifications

### Safety & Security
- File operations: Dry-run with confirmation, path canonicalization, jail to project root
- Error handling: Typed ADTs everywhere for explicit, safe error handling
- Credentials: Environment variables only (standard practice)

### Quality
- Testing: Property tests (hedgehog) for core logic, unit tests for integration (tasty framework)
- Linting: hlint + ormolu for consistent code
- Logging: Four levels (error, warn, info, debug) with warn as default; --debug flag for RPC dumps
  - Dynamic control via setLogLevel RPC method
  - Output: stderr (default) or file via DUET_RPC_LOG
  - Redaction: tokens, secrets, paths automatically removed

## Non-Functional Requirements

 

### Resource Limits
- Max file size: 1MB per file
- Max request size: 10MB (includes headers up to 8KB); 30s read timeout
- Memory budget: <500MB
- Execution: Sequential processing (one request at a time)

### Reliability
- Auto-restart on crash with backoff
- Smart retry for transient errors (3x)
- Read timeout: 30s per message (fixed in v1)
- No data loss via mandatory backups

## Implementation Structure

### Module Organization
```
src/
  DuetRpc/
    CLI/           -- optparse-applicative parsers ([[CLI Parser|Parser Component]])
    RPC/           -- JSON-RPC handling ([[Component Map#RPC Components|RPC Handler]])
    Providers/     -- AI provider client interfaces
    Context/       -- Context assembly utilities
    Session/       -- Session management helpers
    FileOps/       -- Safe file operations
    Config/        -- Configuration ([[Config Loader|Config Component]])
    Logging/       -- Katip setup ([[Logger|Logger Component]])
    Types/         -- Shared types and errors ([[Error Handler|Error Management]])
```

### Error Type Hierarchy
```haskell
data DuetError
  = ConfigError ConfigError
  | RPCError RPCError
  | ProviderError ProviderError
  | FileError FileError
  | SessionError SessionError
  deriving (Show, Eq)
```

### Error Translation
- Standard JSON-RPC 2.0 codes (−32700 Parse, −32600 Invalid Request, −32601 Method Not Found, −32602 Invalid Params, −32603 Internal)
- Minimal error.data with safe fields only; detailed diagnostics to logs
- Automatic redaction of stack traces, paths, tokens, environment values

### Configuration Precedence
1. Command-line flags
2. Environment variables
3. Project `.duet-rpc.toml`
4. User config (`~/.config/duet-rpc/config.toml`)
5. System defaults

## Quality Gates

- All architecture decisions documented and accepted
- NFR targets specified with measurable criteria
- Error handling strategy consistently applied
- Security considerations explicitly addressed
- Testing approach defined with framework choices

---

Notes
- Naming: `codex` refers to OpenAI's CLI; `claude-code` refers to Anthropic's CLI; `duet-rpc` is our RPC/CLI binary used by Emacs and for one-shots, supporting both providers.
- v1 targets Emacs UI + duet-rpc over stdio; no network server required.
- CLI proposes patches; Emacs applies edits after user approval (dry-run by default, allowlist paths, size/file caps).
