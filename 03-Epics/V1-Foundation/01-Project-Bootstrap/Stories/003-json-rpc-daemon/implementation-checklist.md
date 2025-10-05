# Story 003: JSON-RPC Daemon Implementation Checklist

## Overview
This checklist breaks down the implementation of the JSON-RPC daemon into ordered, actionable tasks. Follow KISS and YAGNI principles - implement only what's specified in the story, no future-proofing.

---

## Phase 1: Core Dependencies & Types

### 1.1 Add Required Dependencies
- [ ] Add `aeson` to duet-rpc.cabal for JSON parsing
- [ ] Add `bytestring` to duet-rpc.cabal for binary I/O
- [ ] Add `stm` to duet-rpc.cabal for shutdown coordination (ADR-006)
- [ ] Add `unix` to duet-rpc.cabal for signal handling (SIGINT, SIGTERM, SIGHUP)
- [ ] Add `async` to duet-rpc.cabal for timeout handling

### 1.2 Define JSON-RPC Core Types
- [ ] Create `src/Duet/Rpc/JsonRpc/Types.hs` module
- [ ] Define `JsonRpcRequest` type with fields: `jsonrpc`, `method`, `params`, `id`
  - `jsonrpc`: must be string "2.0"
  - `method`: string
  - `params`: optional Value (from aeson)
  - `id`: custom type that accepts string, number, or null (reject object/array/boolean)
- [ ] Define `JsonRpcResponse` type with fields: `jsonrpc`, `result`, `id`
- [ ] Define `JsonRpcError` type with fields: `jsonrpc`, `error`, `id`
- [ ] Define `JsonRpcErrorDetail` type with fields: `code`, `message`, `data` (optional)
- [ ] Define `RequestId` sum type for valid IDs: `IdString Text | IdNumber Scientific | IdNull`
- [ ] Implement JSON instances (FromJSON/ToJSON) for all types
  - Ensure RequestId rejects object/array/boolean during parsing
  - Ensure RequestId encoding preserves exact type (string stays string, number stays number)

### 1.3 Define Error Data Types
- [ ] Create `src/Duet/Rpc/JsonRpc/ErrorData.hs` module
- [ ] Define `InvalidParamsData` type: `param`, `expected`, `received`, `accepted` (optional list)
- [ ] Define `InvalidRequestReason` enum: `UnsupportedContentType | Oversize | BadCharset | InvalidIdType | BatchNotSupported | HeaderTooLarge`
- [ ] Define `InvalidRequestData` type with `reason` field
- [ ] Implement ToJSON instances for error data types (never include in FromJSON - these are outbound only)

---

## Phase 2: LSP-Style Framing Layer

### 2.1 Implement Frame Parser
- [ ] Create `src/Duet/Rpc/JsonRpc/Framing.hs` module
- [ ] Define `FrameHeader` type with fields: `contentLength :: Int`, `contentType :: Maybe Text`
- [ ] Implement `parseHeaders :: ByteString -> Either FramingError FrameHeader`
  - Parse header lines until `\r\n\r\n` delimiter
  - Header names are case-insensitive (normalize to lowercase for lookup)
  - Extract required `Content-Length` header (error if missing)
  - Extract optional `Content-Type` header
  - Ignore unknown headers
  - Enforce 8 KB cap on entire header section (before `\r\n\r\n`)
- [ ] Implement Content-Type validation in separate function
  - If present, must be `application/vscode-jsonrpc; charset=utf-8`
  - Media type must match exactly (case-insensitive)
  - Charset parameter must be `utf-8` (case-insensitive)
  - Parameter order is not significant
  - Return InvalidRequestReason if validation fails
- [ ] Implement `readFramedMessage :: Handle -> Int -> IO (Either FramingError ByteString)`
  - Check Content-Length <= 10 MB before reading (error with id=null if exceeded)
  - Use timeout (30 seconds) for reading body
  - Validate exact byte count matches Content-Length
  - Return error if EOF before N bytes or excess bytes after

### 2.2 Define Framing Error Types
- [ ] Create framing error ADT: `ParseError | InvalidRequest | ContentLengthMismatch | Timeout | Oversize | HeaderTooLarge`
- [ ] Map framing errors to JSON-RPC error codes per ADR-012:
  - ParseError, ContentLengthMismatch → -32700
  - InvalidRequest, Oversize, HeaderTooLarge → -32600 with appropriate error.data.reason
  - Timeout → discard partial data, log, continue (no error response)

### 2.3 Implement Frame Writer
- [ ] Implement `writeFramedResponse :: Handle -> ByteString -> IO ()`
  - Calculate UTF-8 byte length
  - Write `Content-Length: N\r\n\r\n` header
  - Write JSON response body
  - Flush stdout immediately after each response

---

## Phase 3: JSON-RPC Message Handling

### 3.1 Implement Request Parser
- [ ] Create `src/Duet/Rpc/JsonRpc/Parser.hs` module
- [ ] Implement `parseRequest :: ByteString -> Either JsonRpcError JsonRpcRequest`
  - Decode JSON from UTF-8 bytes
  - Check if result is JSON array → reject with -32600 "Batch requests not supported" and error.data.reason: "batch-not-supported"
  - Check for required `jsonrpc` field = "2.0"
  - Check for required `method` field (string)
  - Parse optional `params` field
  - Parse optional `id` field (use custom RequestId parser that rejects object/array/boolean)
  - Return appropriate JSON-RPC error for validation failures

### 3.2 Implement Response Serializer
- [ ] Implement `serializeResponse :: JsonRpcResponse -> ByteString`
  - Encode response to JSON
  - Convert to UTF-8 bytes
  - Ensure `id` is echoed exactly as received (preserving type)
- [ ] Implement `serializeError :: JsonRpcError -> ByteString`
  - Encode error response to JSON
  - Convert to UTF-8 bytes
  - Use canonical error messages per ADR-012
  - Include minimal error.data per ADR-012

### 3.3 Distinguish Requests vs Notifications
- [ ] Implement `isNotification :: JsonRpcRequest -> Bool`
  - Check if `id` field is absent (not just null - actually missing)
  - Return True if missing, False otherwise
- [ ] Handle notifications in dispatcher:
  - Process method without sending response
  - Log unknown-method notifications at `warn` level
  - Log parameter/internal errors at `error` level
  - Never terminate daemon for notification errors

---

## Phase 4: RPC Method Implementations

### 4.1 Define Method Handler Type
- [ ] Create `src/Duet/Rpc/JsonRpc/Methods.hs` module
- [ ] Define `MethodHandler` type: `Value -> IO (Either JsonRpcError Value)`
  - Takes params (JSON Value), returns result or error
- [ ] Define `MethodRegistry` type: `Map Text MethodHandler`

### 4.2 Implement Core Methods
- [ ] Implement `initialize` method
  - No parameters required
  - Return `{"serverInfo": {"name": "duet-rpc", "version": "VERSION"}, "protocolVersion": "2.0"}`
  - Use Paths_duet_rpc.version to get version dynamically
  - Version must be strict SemVer (no build metadata)

- [ ] Implement `listMethods` method
  - No parameters required
  - Return array of objects: `[{"name": "METHOD", "description": "DESC"}, ...]`
  - Include all 6 methods: initialize, listMethods, describeMethods, version, setLogLevel, shutdown
  - Descriptions are human-readable strings (not formal schemas)

- [ ] Implement `describeMethods` method
  - No parameters required
  - Return array of objects: `[{"name": "METHOD", "params": ["param: type"], "returns": "type"}, ...]`
  - Signatures are human-readable strings like "level: string"
  - Not formal JSON Schema or OpenAPI

- [ ] Implement `version` method
  - No parameters required
  - Return `{"version": "VERSION"}` using Paths_duet_rpc.version
  - Version must be strict SemVer (no build metadata)

### 4.3 Implement setLogLevel Method
- [ ] Accept params object with `level` field (string)
- [ ] Normalize level to lowercase
- [ ] Validate against accepted values: debug, info, warn, error (case-insensitive)
- [ ] Return error -32602 with error.data showing accepted values if invalid
- [ ] Update logger severity dynamically using Katip's runtime configuration
- [ ] Return `{"level": "NORMALIZED_LEVEL", "success": true}` on success

### 4.4 Implement shutdown Method
- [ ] Accept no parameters (params can be omitted or null)
- [ ] Set shutdown flag in shared state (use STM TVar per ADR-006)
- [ ] Return `{"message": "Shutting down gracefully"}`
- [ ] Trigger graceful shutdown sequence after sending response
- [ ] Support as notification (no response sent, same shutdown behavior)

---

## Phase 5: Request Dispatcher

### 5.1 Implement Main Dispatch Loop
- [ ] Create `src/Duet/Rpc/JsonRpc/Dispatcher.hs` module
- [ ] Implement single-threaded sequential dispatcher per ADR-011
  - Read one framed message from stdin
  - Parse JSON-RPC request
  - Check if notification (no response needed)
  - Look up method in registry
  - Execute handler
  - Serialize and write response (if not notification)
  - Repeat until shutdown or EOF

### 5.2 Error Handling in Dispatcher
- [ ] Map method lookup failure to -32601 "Method not found"
- [ ] Map handler exceptions to -32603 "Internal error"
- [ ] Ensure daemon stays alive after -32603 errors
- [ ] Log all errors with appropriate severity per ADR-014
- [ ] Include `method` and `id` fields in logs when available

### 5.3 Response Ordering
- [ ] Ensure responses are written in same order as requests received
  - Single-threaded execution naturally preserves order
  - No additional coordination needed for v1

---

## Phase 6: Daemon Lifecycle & Signals

### 6.1 Implement Shutdown Coordination
- [ ] Create `src/Duet/Rpc/JsonRpc/Lifecycle.hs` module
- [ ] Define shutdown state using STM TVar Bool
- [ ] Implement `initiateShutdown :: TVar Bool -> IO ()`
  - Set TVar to True
  - Stop reading new frames
- [ ] Implement graceful shutdown with 2-second deadline
  - If current request is `shutdown`, reply then exit
  - If other request running, try to finish and send response
  - Exit without response if not done within 2 seconds

### 6.2 Implement Signal Handlers
- [ ] Install handlers for SIGINT, SIGTERM, SIGHUP
- [ ] All signals trigger same graceful shutdown
- [ ] Log signal name at `info` level when received
- [ ] Use System.Posix.Signals.installHandler

### 6.3 Implement EOF Handling
- [ ] Detect EOF on stdin in main read loop
- [ ] Log "stdin closed, shutting down gracefully" at `info` level
- [ ] Trigger graceful shutdown (exit 0 within 2 seconds)

### 6.4 Implement Cleanup & Flush
- [ ] Flush stdout before exit
- [ ] Flush and close Katip log scribes
- [ ] Cap log flush at 500ms (exit anyway to meet 2-second deadline)
- [ ] Use timeout mechanism for flush operations

### 6.5 Define Exit Codes
- [ ] Exit 0 for graceful shutdown (EOF, signals, shutdown request/notification)
- [ ] Exit 1 for unrecoverable errors (initialization failure, fatal IO errors)

---

## Phase 7: Logging Integration

### 7.1 Enhance Logger Module
- [ ] Update `src/Duet/Rpc/Logger.hs` to support RPC-specific structured logging
- [ ] Add logging functions that accept context fields: `method`, `id`, `severity`
- [ ] Implement startup logging:
  - Log version (from Paths_duet_rpc)
  - Log PID (from System.Posix.Process.getProcessID)
  - Log active log level
  - Log sink (stderr or file path from DUET_RPC_LOG)
- [ ] Ensure all logs include minimum fields per ADR-014: timestamp, severity, method (when known), id (when parseable), message

### 7.2 Implement Severity Guidance
- [ ] Client-caused errors (parse, invalid-request, invalid-params, unknown-method) → log at `warn`
- [ ] Internal faults (uncaught exceptions, -32603 errors) → log at `error`
- [ ] Lifecycle events (startup, shutdown, signals, EOF) → log at `info`
- [ ] Debug information (frame received, request dispatched) → log at `debug`

### 7.3 Implement Redaction
- [ ] Never log payload bodies for parse errors or oversize errors
- [ ] Never log secrets, tokens, or user paths
- [ ] Log only safe metadata: method name, id, error codes, sizes, timing

### 7.4 Respect Color Flags
- [ ] Honor `--no-color` CLI flag (already in CliOptions)
- [ ] Honor `NO_COLOR` environment variable (check in Logger.initLogger)
- [ ] Disable color when not TTY
- [ ] File sink always uses plain text (no ANSI codes)

---

## Phase 8: RPC Command Integration

### 8.1 Update CLI Shell
- [ ] Modify `src/Duet/Rpc/CLI/Shell.hs`
- [ ] Replace `runRpc` placeholder with actual RPC daemon launcher
- [ ] Pass CliOptions to RPC daemon (for log level, no-color flag)

### 8.2 Implement RPC Daemon Entry Point
- [ ] Create `src/Duet/Rpc/Rpc/Daemon.hs` module
- [ ] Implement `runRpcDaemon :: CliOptions -> IO ()`
  - Initialize logger with startup logging
  - Check DUET_RPC_LOG environment variable
  - Install signal handlers
  - Initialize shutdown state (TVar)
  - Build method registry
  - Start dispatcher loop reading from stdin
  - Handle EOF gracefully
  - Clean up and exit with appropriate code

### 8.3 Stdout Discipline
- [ ] Verify logger never writes to stdout (already configured for stderr/file)
- [ ] Ensure only framed JSON-RPC responses go to stdout
- [ ] Test that non-JSON output breaks client parsers (this should never happen)

---

## Phase 9: Testing

### 9.1 Unit Tests for Framing
- [ ] Create `test/Duet/Rpc/Test/Unit/Framing.hs`
- [ ] Test valid Content-Length header parsing
- [ ] Test Content-Type validation (valid, invalid media type, invalid charset)
- [ ] Test header case-insensitivity
- [ ] Test unknown headers are ignored
- [ ] Test 8 KB header cap (exceed → error -32600 with error.data.reason: "header-too-large")
- [ ] Test 10 MB request size limit (exceed → error -32600 with id: null, error.data.reason: "oversize")
- [ ] Test Content-Length mismatch (EOF before N bytes, excess after N bytes → error -32700)
- [ ] Test 30-second read timeout (timeout → discard partial, log, continue)

### 9.2 Unit Tests for JSON-RPC Parsing
- [ ] Create `test/Duet/Rpc/Test/Unit/JsonRpc.hs`
- [ ] Test valid request parsing
- [ ] Test batch array rejection (error -32600 with message "Batch requests not supported" and error.data.reason: "batch-not-supported")
- [ ] Test missing jsonrpc field (error -32600)
- [ ] Test invalid jsonrpc value (not "2.0" → error -32600)
- [ ] Test missing method field (error -32600)
- [ ] Test request ID types:
  - String, number, null → accepted and echoed exactly
  - Object, array, boolean → rejected with error -32600 and error.data.reason: "invalid-id-type"
- [ ] Test notification detection (missing id field, not just null)

### 9.3 Unit Tests for Methods
- [ ] Create `test/Duet/Rpc/Test/Unit/Methods.hs`
- [ ] Test `initialize` returns correct structure with actual version
- [ ] Test `listMethods` returns all 6 methods with descriptions
- [ ] Test `describeMethods` returns all 6 methods with signatures
- [ ] Test `version` returns correct version
- [ ] Test `setLogLevel`:
  - Valid levels (debug, info, warn, error) → success with normalized level
  - Case variations (DEBUG, Info) → accepted and normalized
  - Invalid level → error -32602 with error.data showing accepted values
- [ ] Test `shutdown` returns graceful shutdown message

### 9.4 Integration Tests for Protocol Behavior
- [ ] Create `test/Duet/Rpc/Test/Integration/Protocol.hs`
- [ ] Test single framed request/response
- [ ] Test multiple consecutive framed messages (processed in order)
- [ ] Test notifications (no response sent)
- [ ] Test unknown-method notification (logged at warn, no response, daemon continues)
- [ ] Test shutdown notification (triggers graceful exit without response)
- [ ] Test response ordering matches request order
- [ ] Test method not found (error -32601)
- [ ] Test invalid params (error -32602 with error.data structure)
- [ ] Test internal error (error -32603, daemon stays alive)

### 9.5 Integration Tests for Lifecycle
- [ ] Create `test/Duet/Rpc/Test/Integration/Lifecycle.hs`
- [ ] Test stdin EOF triggers graceful shutdown (exit 0 within 2 seconds)
- [ ] Test SIGINT triggers graceful shutdown (log signal name, exit 0)
- [ ] Test SIGTERM triggers graceful shutdown (log signal name, exit 0)
- [ ] Test SIGHUP triggers graceful shutdown (log signal name, exit 0)
- [ ] Test shutdown request during in-flight request:
  - If current request is shutdown → reply then exit
  - If other request running → attempt to finish, exit without response if exceeds 2 seconds
- [ ] Test log flush with 500ms cap (exit anyway if incomplete)

### 9.6 Integration Tests for Logging
- [ ] Create `test/Duet/Rpc/Test/Integration/Logging.hs`
- [ ] Test startup logging (version, PID, level, sink)
- [ ] Test log fields present (timestamp, severity, method, id, message)
- [ ] Test severity guidance (client errors → warn, internal → error, lifecycle → info)
- [ ] Test redaction (no payload bodies, no secrets, no paths)
- [ ] Test `--no-color` flag disables color
- [ ] Test `NO_COLOR` env var disables color
- [ ] Test `DUET_RPC_LOG` file sink (writable → use file, not writable → warn and use stderr)
- [ ] Test dynamic log level change via `setLogLevel` RPC method

### 9.7 Integration Tests for Error Data Structure
- [ ] Test -32602 error.data includes: param, expected, received, accepted (when applicable)
- [ ] Test -32600 error.data includes: reason (unsupported-content-type, oversize, bad-charset, invalid-id-type, batch-not-supported, header-too-large)
- [ ] Test error responses never include stack traces, paths, tokens, or env values

---

## Phase 10: Documentation & Validation

### 10.1 Update Documentation
- [ ] Add docstrings to all public functions and types
- [ ] Document error codes and error.data structures
- [ ] Document method signatures and parameters
- [ ] Document resource limits (10 MB, 8 KB, 30s)

### 10.2 Validation Against Story Requirements
- [ ] Verify all 6 RPC methods work correctly
- [ ] Verify all 5 error types (-32700, -32600, -32601, -32602, -32603) with correct error.data
- [ ] Verify protocol edge cases (batch, notifications, ID semantics, framing)
- [ ] Verify resource limits (10 MB request, 8 KB header, 30s timeout, partial frame discard)
- [ ] Verify lifecycle (EOF, SIGINT, SIGTERM, SIGHUP, in-flight handling, flush timeout)
- [ ] Verify logging (startup, severity, fields, redaction, color control)
- [ ] Verify stdout discipline (only framed JSON-RPC responses)
- [ ] Verify compatibility with Emacs jsonrpc.el (manual test recommended)

### 10.3 Final Checks
- [ ] Run full test suite: `cabal test`
- [ ] Verify exit codes (0 for graceful, 1 for unrecoverable)
- [ ] Verify graceful shutdown always completes within 2 seconds
- [ ] Verify daemon never hangs or blocks indefinitely
- [ ] Check HLint for code quality suggestions
- [ ] Verify all compiler warnings are addressed

---

## Implementation Notes

### Ordering Rationale
1. **Dependencies & Types First**: Foundation for everything else
2. **Framing Layer**: I/O boundary that everything depends on
3. **JSON-RPC Messages**: Protocol layer built on framing
4. **Methods**: Business logic requires protocol support
5. **Dispatcher**: Orchestrates methods and protocol
6. **Lifecycle**: Requires dispatcher and methods in place
7. **Logging**: Cross-cutting concern integrated throughout
8. **CLI Integration**: Wires everything together
9. **Testing**: Validates each layer and integration
10. **Documentation**: Final polish and validation

### Key Principles
- **KISS**: Implement exactly what's in the story, no extras
- **YAGNI**: Don't build for future requirements (e.g., no concurrent dispatcher, no configurable limits)
- **Fail Gracefully**: Daemon stays alive for client errors, logs and reports properly
- **Testability**: Each component is tested in isolation, then integration

### Critical Requirements
- Stdout discipline: ONLY framed JSON-RPC responses on stdout
- Sequential processing: No concurrent requests in v1
- Graceful shutdown: Always within 2 seconds
- Error data: Minimal, non-sensitive, per ADR-012
- Logging: Structured, redacted, never to stdout
