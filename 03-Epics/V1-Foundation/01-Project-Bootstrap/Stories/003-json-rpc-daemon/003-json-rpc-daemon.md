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

### Protocol Requirements (Must Have)

- **Transport**: JSON-RPC 2.0 over stdio with LSP-style framing
  - Format: `Content-Length: {bytes}\r\n\r\n{json-rpc-message}`
  - Encoding: UTF-8 only
  - Content-Type (optional): `application/vscode-jsonrpc; charset=utf-8` (charset case-insensitive)
    - Other media types/charsets return error -32600
    - If omitted, assumes default
  - Content-Length (required): Must match UTF-8 byte count
    - Max size: 10 MB (exceeding returns error -32600)
    - Mismatch returns error -32700
    - Read timeout: 30s (incomplete reads logged, partial data discarded, daemon continues)
  - Unknown headers: Ignored
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

- **CLI Compatibility**: `duet-rpc rpc` accepts existing global flags (`--log-level`, `--no-color`, `DUET_RPC_LOG`)

- **Stdout Discipline**: stdout carries ONLY framed JSON-RPC responses
  - All logs/diagnostics go to stderr or log file (if `DUET_RPC_LOG` set)
  - Non-JSON output on stdout breaks client parsers

- **Lifecycle**:
  - stdin EOF: Graceful shutdown (exit 0 within 2s), logged at `info`
  - Signals (SIGINT/SIGTERM/SIGHUP): Graceful shutdown (exit 0 within 2s), logged at `info`
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
- **-32700 Parse Error**: Invalid JSON
- **-32600 Invalid Request**: Malformed JSON-RPC, unsupported Content-Type, or size > 10 MB
- **-32601 Method Not Found**: Unknown method
- **-32602 Invalid Params**: Wrong types or missing required fields
- **-32603 Internal Error**: Unhandled daemon errors (daemon stays alive)

**Error Data**: Optional `error.data` may contain param name, expected/received values. No stack traces, paths, secrets, or env details.

## Acceptance Criteria (Given/When/Then)

### Daemon Lifecycle
- `duet-rpc rpc` starts daemon, listens on stdin, runs until `shutdown` request
- Unhandled errors: log to stderr, return JSON-RPC error, stay alive
- `shutdown` request: returns `{"message": "Shutting down gracefully"}`, exits 0 within 2s

### Protocol Framing
- `Content-Length: N\r\n\r\n{json}` with matching byte count: parses correctly
- Multiple consecutive/back-to-back framed messages: processes independently in order
- `Content-Type: application/vscode-jsonrpc; charset=utf-8` and charset variations (`UTF-8`, `utf-8`): accepted
- Wrong Content-Type (`application/json`) or charset (`iso-8859-1`): error -32600
- Content-Length mismatch: error -32700
- Content-Length > 10 MB: error -32600
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
- Batch requests (JSON array): error -32600 with message "Batch requests not supported"
- Notifications (no `id`): processed silently, no response; unknown methods logged at `warn`
- Request ID: string/number/null echoed exactly; object/array returns error -32600
- Response ordering: same order as requests received

### Lifecycle & Signals
- stdin EOF: logs "stdin closed, shutting down gracefully" at `info`, exits 0 within 2s
- SIGINT/SIGTERM: logs signal at `info`, exits 0 within 2s

### Error Cases
- Invalid JSON: error -32700
- Missing `jsonrpc` field: error -32600
- Unknown method: error -32601
- Missing params: error -32602

### CLI Flags
- `--log-level debug`: debug logs on stderr
- `DUET_RPC_LOG=/tmp/rpc.log`: logs to file

### Testing
- All 6 methods validated with functional tests
- All 5 error types validated
- Protocol edge cases: batch, notifications, ID semantics, framing
- Lifecycle: EOF, SIGINT, SIGTERM

## Assumptions
- Single-threaded request handling sufficient (high confidence; defer queueing to future if needed)

## Success Metrics
- All 6 RPC methods respond correctly in automated tests
- All error scenarios return spec-compliant responses
- Lifecycle validated in CI (startup, shutdown, EOF, signals)
- Protocol compliance validated (batch, notifications, ID, framing, ordering)
- Resource limits enforced (10 MB, 30s timeout)

## Related Documentation
- [[Stories/001-cli-version-help-bootstrap/001-cli-version-help-bootstrap|Story 001]] - CLI foundation
- [[01-Architecture/duet-rpc/ADRs/ADR-002 RPC Protocol|ADR-002]] - JSON-RPC protocol choice
- [[01-Architecture/duet-rpc/duet-rpc Technical Architecture|Technical Architecture]] - RPC design context

---
*Navigation: [[00-Start-Here/README|Home]] > [[03-Epics]] > [[03-Epics/V1-Foundation/README|V1-Foundation]] > [[03-Epics/V1-Foundation/01-Project-Bootstrap/README|Project Bootstrap]] > Story 003*
