---
tags: [story, rpc, daemon, json-rpc, foundation]
aliases: [Story 003, JSON-RPC Daemon, RPC Interface Story]
---
# Story 003: JSON-RPC Daemon Interface

As a duet-rpc developer, I want a `duet-rpc rpc` command that launches a long-running daemon with a JSON-RPC 2.0 interface so that Emacs can communicate with the CLI over stdio.

## Priority
- Must

## Dependencies
- Story 001 (CLI Version/Help Bootstrap) - Complete

## Non-Goals
- Emacs client implementation (deferred to Story 004)
- Implementation of `doctor` or `prompt` command placeholders
- AI provider integration or streaming responses
- File operations or context building
- Session state management beyond basic daemon lifecycle
- Multi-session or concurrent request handling
- Network transport (stdio only)

## Business Rationale
- Enables programmatic access to duet-rpc functionality for Emacs integration
- Provides discoverable, testable RPC interface that serves as foundation for all future Emacs workflows
- Establishes protocol compliance with JSON-RPC 2.0 standard for interoperability
- Supports graceful daemon lifecycle management (startup, configuration, shutdown)

## Scope

### RPC Methods (Must Have)

The following methods expose Story 001 CLI functionality via JSON-RPC:

1. **`initialize`** - Returns server metadata
   - Params: None
   - Returns: `{"serverInfo": {"name": "duet-rpc", "version": "0.1.0"}, "protocolVersion": "2.0"}`
   - Version format: Strict SemVer (e.g., "0.1.0"), no build metadata

2. **`listMethods`** - Returns available RPC methods with descriptions
   - Params: None
   - Returns: Array of `{"name": "string", "description": "string"}`
   - Descriptions are human-readable only, not formal schemas (JSON Schema/OpenAPI)

3. **`describeMethods`** - Returns method signatures
   - Params: None
   - Returns: Array of `{"name": "string", "params": ["param: type"], "returns": "type"}`
   - Signatures are human-readable strings (e.g., "level: string"), not formal schemas

4. **`version`** - Returns version information
   - Params: None
   - Returns: `{"version": "0.1.0"}` (strict SemVer, no build metadata)

5. **`setLogLevel`** - Dynamically adjusts daemon log verbosity
   - Params: `{"level": "debug|info|warn|error"}` (case-insensitive)
   - Returns: `{"level": "debug", "success": true}`
   - Invalid values return error -32602 with `error.data` showing accepted values

6. **`shutdown`** - Gracefully terminates the daemon
   - Params: None
   - Returns: `{"message": "Shutting down gracefully"}`
   - Also supported as notification (no response sent, follows same 2-second shutdown window)

### Protocol Requirements (Must Have)

- **Transport**: JSON-RPC 2.0 over stdio with LSP-style framing
  - Format: `Content-Length: {bytes}\r\n\r\n{json-rpc-message}`
  - Encoding: UTF-8 only
  - Content-Type (optional): `application/vscode-jsonrpc; charset=utf-8` (charset case-insensitive)
    - Other media types/charsets return error -32600
    - If omitted, assumes default
  - Content-Length (required): Must match UTF-8 byte count
    - Max size: 10 MB (exceeding returns error -32600 with `id: null`, payload discarded without reading)
    - Mismatch returns error -32700
    - Read timeout: 30s per message (incomplete reads logged, partial data discarded, framing state reset, daemon continues; multiple timeouts do not terminate daemon)
  - Header cap: 8 KB for header section (exceeding returns error -32600 with `id: null`, discarded, daemon continues)
  - Unknown headers: Ignored
  - Partial frames: Never buffered across timeouts; framing state reset after timeout
  - Compatible with Emacs jsonrpc.el

- **Method Invocation**: Any method callable immediately; no initialization sequence required

- **Batch Requests**: Not supported
  - JSON array requests return error -32600 with message "Batch requests not supported"

- **Notifications** (requests without `id` field): Supported
  - Valid notifications processed silently, no response sent
  - Unknown-method notifications logged at `warn` level, daemon continues

- **Request ID Semantics**:
  - Accepted types: `string`, `number`, or `null`
  - Other types (object, array, boolean) rejected with error -32600
  - ID echoed exactly as received (preserves type and value)
  - Responses sent in same order as requests received

- **Concurrency**: Single-threaded sequential processing
  - Requests handled one at a time in order
  - Clients should not pipeline concurrent requests

- **Logging & Diagnostics**:
  - Logger: Katip
  - Sinks: Default stderr; if `DUET_RPC_LOG` points to writable file, log there (checked at startup; on failure, warn to stderr and fall back to stderr)
  - Levels: `debug|info|warn|error` (case-insensitive, normalized to lowercase)
  - Formatting & color: stderr uses human-readable format with color only when TTY and color enabled; honor `--no-color` flag and `NO_COLOR` env var; file sink uses plain text
  - Log fields (minimum): `timestamp`, `severity`, `method` (when known), `id` (when parseable, including null; omit if unparseable), `message`
  - Severity guidance: client-caused errors (parse/invalid-request/invalid-params/unknown-method) → `warn`; internal faults → `error`; lifecycle events → `info`
  - Privacy/redaction: do not log payload bodies on parse/oversize failures; never log secrets, tokens, or user paths
  - Startup logging: Log version, PID, active level, and sink (stderr or file path) at startup

- **CLI Compatibility**: `duet-rpc rpc` accepts existing global flags (`--log-level`, `--no-color`) and env vars (`DUET_RPC_LOG`, `NO_COLOR`)

- **Stdout Discipline**: stdout carries ONLY framed JSON-RPC responses
  - All logs/diagnostics go to stderr or log file (if `DUET_RPC_LOG` set)
  - Non-JSON output on stdout breaks client parsers

- **Lifecycle**:
  - stdin EOF: Graceful shutdown (exit 0 within 2s), logged at `info` as "stdin closed, shutting down gracefully"
  - Signals (SIGINT/SIGTERM/SIGHUP): Graceful shutdown (exit 0 within 2s), logged at `info` with signal name
  - Shutdown behavior:
    - Stop reading new frames immediately upon trigger
    - If current request is `shutdown`, reply then exit
    - If non-shutdown request running, attempt to finish and send response; if not done by 2s, exit without response
    - Flush stdout and logs; cap log flush at 500ms; exit anyway if incomplete to meet 2s deadline
  - Framing errors: Log to stderr, send error response if possible, continue
  - Unrecoverable errors: Exit code 1

- **Security**: Local-only stdio communication with parent process. Trust boundary is parent process - daemon trusts all stdin input. No authentication. Same privileges as parent. Not for untrusted input.

- **Client Requirements**:
  - Properly framed messages with accurate `Content-Length` (UTF-8 byte count)
  - UTF-8 encoded JSON payloads
  - If using `Content-Type`, must be `application/vscode-jsonrpc; charset=utf-8` (or omit)
  - No batch requests (returns error -32600)
  - `initialize` optional but recommended
  - Do not write to daemon stdout
  - No concurrent request pipelining (single-threaded)

### Error Handling (Must Have)

JSON-RPC 2.0 error codes:
- **-32700 Parse Error**: Invalid JSON or Content-Length mismatch
- **-32600 Invalid Request**: Malformed JSON-RPC envelope, unsupported Content-Type/charset, request size > 10 MB, header size > 8 KB, invalid id type, batch array
- **-32601 Method Not Found**: Unknown method
- **-32602 Invalid Params**: Wrong types or missing required fields
- **-32603 Internal Error**: Unexpected server faults (daemon stays alive)

**Error Response `id`**: Echo request `id` exactly when parseable; use `id: null` if unparseable or size exceeded before reading.

**Error Data Structure** (minimal, non-sensitive):
- For **-32602**: `{ "param": "...", "expected": "...", "received": "...", "accepted": ["..."]? }`
- For **-32600**: `{ "reason": "unsupported-content-type|oversize|bad-charset|invalid-id-type|batch-not-supported|header-too-large" }`
- Never include stack traces, filesystem paths, tokens, or environment values

## Acceptance Criteria (Given/When/Then)

### Daemon Lifecycle
- `duet-rpc rpc` starts daemon, listens on stdin, runs until `shutdown` request
- Unhandled errors: log to stderr, return JSON-RPC error, stay alive
- `shutdown` request: returns `{"message": "Shutting down gracefully"}`, exits 0 within 2s

### Protocol Framing
- `Content-Length: N\r\n\r\n{json}` with matching byte count: parses correctly
- Multiple consecutive/back-to-back framed messages: processes independently in order
- `Content-Type: application/vscode-jsonrpc; charset=utf-8` and charset variations (`UTF-8`, `utf-8`): accepted
- Wrong Content-Type (`application/json`) or charset (`iso-8859-1`): error -32600 with `error.data.reason: "unsupported-content-type|bad-charset"`
- Content-Length mismatch: error -32700
- Content-Length > 10 MB: error -32600 with `id: null`, `error.data.reason: "oversize"`, payload not read
- Header section > 8 KB: error -32600 with `id: null`, `error.data.reason: "header-too-large"`
- Read timeout after 30s: partial data discarded, framing state reset, daemon continues
- Unknown headers: ignored

### Methods
- `initialize`: returns `{"serverInfo": {"name": "duet-rpc", "version": "0.1.0"}, "protocolVersion": "2.0"}` with actual version
- `listMethods`: returns array of 6 methods with `{"name": "...", "description": "..."}`
- `describeMethods`: returns array of 6 methods with params/return types
- `version`: returns `{"version": "0.1.0"}` with actual version
- `setLogLevel`:
  - Valid: returns `{"level": "debug", "success": true}`, updates log level
  - Case variations (`DEBUG`, `Info`): accepted and normalized
  - Invalid level: error -32602 with `error.data` showing accepted values

### Protocol Behavior
- Batch requests (JSON array): error -32600 with message "Batch requests not supported" and `error.data.reason: "batch-not-supported"`
- Notifications (no `id`): processed silently, no response; unknown methods logged at `warn`
- `shutdown` notification: triggers graceful exit without response, follows same 2-second window
- Request ID: string/number/null echoed exactly; object/array/boolean returns error -32600 with `error.data.reason: "invalid-id-type"`
- Response ordering: same order as requests received

### Lifecycle & Signals
- stdin EOF: logs "stdin closed, shutting down gracefully" at `info`, exits 0 within 2s
- SIGINT/SIGTERM/SIGHUP: logs signal name at `info`, exits 0 within 2s
- In-flight request during shutdown: attempts to finish and send response; if not done by 2s, exits without response
- Log flush: capped at 500ms; exits anyway if incomplete to meet 2s deadline

### Error Cases
- Invalid JSON: error -32700
- Missing `jsonrpc` field: error -32600
- Unknown method: error -32601
- Missing params: error -32602
- Internal error: error -32603

### CLI Flags & Environment
- `--log-level debug`: debug logs on stderr
- `DUET_RPC_LOG=/tmp/rpc.log`: logs to file (checks writability at startup, falls back to stderr on failure with warning)
- `--no-color`: disables color output
- `NO_COLOR` env var: disables color output
- Startup logging: logs version, PID, active level, and sink (stderr or file path)
- Log fields: includes timestamp, severity, method (when known), id (when parseable)

### Testing
- All 6 methods validated with functional tests
 - All 6 error types validated with error.data structure verification
- Protocol edge cases: batch, notifications (including shutdown notification), ID semantics, framing
- Resource limits: 10 MB request size, 8 KB header cap, 30s read timeout, partial frame discard
- Lifecycle: EOF, SIGINT, SIGTERM, SIGHUP, in-flight request handling, log flush timeout
- Logging: startup logging, severity levels, field structure, redaction, --no-color, NO_COLOR

## Assumptions
- Single-threaded request handling sufficient (high confidence; defer queueing to future if needed)

## Success Metrics
- All 6 RPC methods respond correctly in automated tests
- All error scenarios return spec-compliant responses with correct error.data structure
- Lifecycle validated in CI (startup, shutdown, EOF, signals, in-flight handling, log flush)
- Protocol compliance validated (batch, notifications including shutdown notification, ID, framing, ordering)
- Resource limits enforced (10 MB request size, 8 KB header cap, 30s timeout, partial frame discard)
- Logging validated (startup logging, severity guidance, field structure, redaction, color control)

## Related Documentation
- [[Stories/001-cli-version-help-bootstrap/001-cli-version-help-bootstrap|Story 001]] - CLI foundation
- [[01-Architecture/duet-rpc/ADRs/ADR-002 RPC Protocol|ADR-002]] - JSON-RPC protocol choice
- [[01-Architecture/duet-rpc/ADRs/ADR-010 RPC IO Framing & Stdout Discipline|ADR-010]] - RPC I/O framing & stdout discipline
- [[01-Architecture/duet-rpc/ADRs/ADR-011 Request Semantics & Concurrency v1|ADR-011]] - Request semantics & concurrency
- [[01-Architecture/duet-rpc/ADRs/ADR-012 Error Translation & Redaction|ADR-012]] - Error translation & redaction
- [[01-Architecture/duet-rpc/ADRs/ADR-013 Daemon Lifecycle & Signals|ADR-013]] - Daemon lifecycle & signals
- [[01-Architecture/duet-rpc/ADRs/ADR-014 Logging & Diagnostics Policy|ADR-014]] - Logging & diagnostics policy
- [[01-Architecture/duet-rpc/ADRs/ADR-015 Resource Limits & Timeouts|ADR-015]] - Resource limits & timeouts
- [[01-Architecture/duet-rpc/duet-rpc Technical Architecture|Technical Architecture]] - RPC design context

---
*Navigation: [[00-Start-Here/README|Home]] > [[03-Epics]] > [[03-Epics/V1-Foundation/README|V1-Foundation]] > [[03-Epics/V1-Foundation/01-Project-Bootstrap/README|Project Bootstrap]] > Story 003*
