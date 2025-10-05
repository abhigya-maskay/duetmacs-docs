---
tags: [architecture, overview, system-design]
aliases: [System Architecture]
---

# Architecture Overview

This document outlines a high-level, three-piece architecture and the responsibilities of each component.

## 1) Emacs UI (ELisp)
- Responsibilities: Editor UX, keybindings, command palette, context capture (region/buffer/file/project), status/progress display, diff review, approval flow, and applying accepted patches to buffers/files.
- Process Lifecycle: Spawn/monitor duet-rpc as a long-lived subprocess; handle version handshake and graceful restarts.
- Configuration: Surface project/global settings (model, presets, ignores) and quick toggles; store lightweight UI preferences.

## 2) RPC CLI Core (duet-rpc, Stdio RPC)
- Responsibilities: Session orchestration, prompt assembly, context building (search, globs, .gitignore), provider calls (streaming), token/rate tracking, diff/patch generation, and safety checks.
- Tools & Feedback: Run tests/linters/build commands; capture errors and feed them back into prompts.
- Persistence: Manage session transcripts, caches, and configuration resolution (global → project) without writing source files directly.
- Interface: JSON-RPC over stdio with streaming events (status/tokens/patch proposals/tool output). One-shot commands also supported for scripts/CI.
- Protocol: JSON-RPC 2.0 with LSP-style framing (Content-Length header); UTF-8 encoding only; stdout carries only JSON-RPC responses, logs to stderr or DUET_RPC_LOG. 10 MB max request size, 8 KB header cap, 30s read timeout.
- Execution: Sequential request processing (one at a time); no batching.

## 3) Adapters/Services
- MCP Adapter: Expose selected tools/providers via MCP for cross-client integrations; reuse CLI logic behind a standard interface.
- Local Daemon/Socket: Background service to share caches/indexes across editor instances.
- HTTP/gRPC Endpoint: For multi-app or remote workflows; introduce security and ops considerations.

## 4) Error Handling & Recovery
- Reconnection: Automatic restart with exponential backoff (1s, 2s, 4s... max 30s) on duet-rpc crash.
- Graceful Degradation: Show clear status when duet-rpc unavailable; allow read-only operations and manual recovery.
- Error Codes: Standard JSON-RPC 2.0 codes; sensitive data redacted from responses.
- Daemon Shutdown: Graceful exit on SIGINT/SIGTERM/SIGHUP/EOF within 2s; exit code 0 (graceful) or 1 (fatal errors).

## 5) Observability
- Structured Logging: Support debug/info/warn/error levels with consistent format; configurable per-component (Emacs UI, duet-rpc).

## 6) Security & Trust Boundaries
- Input Sanitization: Validate file paths and commands; prevent directory traversal and command injection.
- Secrets Handling: API keys never logged or cached; use secure environment variables or keychain integration.

---

Notes
- Naming: `codex` refers to OpenAI's CLI; `claude-code` refers to Anthropic's CLI; `duet-rpc` is our RPC/CLI binary used by Emacs and for one-shots, supporting both providers.
- v1 targets Emacs UI + duet-rpc over stdio; no network server required.
- CLI proposes patches; Emacs applies edits after user approval (dry-run by default, allowlist paths, size/file caps).
