# Story 003: JSON-RPC Daemon Implementation Checklist

## Overview
This checklist breaks down the implementation of the JSON-RPC daemon into ordered, actionable tasks. Follow KISS and YAGNI principles - implement only what's specified in the story, no future-proofing.

## Architecture Principle: Imperative Shell, Functional Core

All implementation must follow the **imperative shell, functional core** pattern:

- **Functional Core (Pure)**: Parsing, validation, transformation, business logic
  - All core logic is pure functions (no IO, no exceptions, explicit errors)
  - Type signatures like `ByteString -> Either Error Result`
  - Testable without IO, deterministic, composable

- **Imperative Shell (Impure)**: I/O operations at system boundaries
  - Handle reading/writing stdin/stdout/stderr
  - Signal handling, file operations, logging
  - Call pure functions from core, handle effects
  - Type signatures include `IO` monad

**Benefits**: Maximizes testability, makes concurrency safer, keeps complexity at edges.

---

## Phase 1: Core Dependencies & Types

### 1.1 Add Required Dependencies
- [x] Add `aeson` to duet-rpc.cabal for JSON parsing
- [x] Add `bytestring` to duet-rpc.cabal for binary I/O
- [x] Add `stm` to duet-rpc.cabal for shutdown coordination (ADR-006)
- [x] Add `unix` to duet-rpc.cabal for signal handling (SIGINT, SIGTERM, SIGHUP)
- [x] Add `async` to duet-rpc.cabal for timeout handling

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

### 2.1 Define Framing Error Types (Functional Core)
- [ ] Create `src/Duet/Rpc/JsonRpc/Framing/Types.hs` module
- [ ] Define `FrameHeader` type with fields: `contentLength :: Int`, `contentType :: Maybe Text`
- [ ] Define framing error ADT: `FramingError`
  - `ParseError Text` - Invalid header format
  - `InvalidContentType Text` - Wrong media type or charset
  - `ContentLengthMismatch Int Int` - Expected vs actual bytes
  - `Oversize Int` - Content-Length exceeds limit
  - `HeaderTooLarge Int` - Header section exceeds cap
  - `MissingContentLength` - Required header absent
- [ ] Define pure mapping function: `framingErrorToJsonRpc :: FramingError -> (Int, Text, Maybe Value)`
  - Returns (error_code, message, error.data)
  - ParseError, ContentLengthMismatch → -32700
  - InvalidContentType, Oversize, HeaderTooLarge → -32600 with error.data.reason

### 2.2 Implement Pure Frame Parsing (Functional Core)
- [ ] Create `src/Duet/Rpc/JsonRpc/Framing/Parser.hs` module
- [ ] Implement `parseHeaders :: ByteString -> Either FramingError FrameHeader` (pure)
  - Parse header lines until `\r\n\r\n` delimiter
  - Header names are case-insensitive (normalize to lowercase for lookup)
  - Extract required `Content-Length` header (error if missing)
  - Extract optional `Content-Type` header
  - Ignore unknown headers
  - Enforce 8 KB cap on entire header section (before `\r\n\r\n`)
- [ ] Implement `validateContentType :: Maybe Text -> Either FramingError ()` (pure)
  - If present, must be `application/vscode-jsonrpc; charset=utf-8`
  - Media type must match exactly (case-insensitive)
  - Charset parameter must be `utf-8` (case-insensitive)
  - Parameter order is not significant
- [ ] Implement `validateContentLength :: Int -> Either FramingError ()` (pure)
  - Check <= 10 MB limit
  - Return `Oversize` error if exceeded
- [ ] Implement `validateBodyLength :: Int -> Int -> Either FramingError ()` (pure)
  - Check expected vs actual byte count
  - Return `ContentLengthMismatch` if mismatch

### 2.3 Implement Pure Frame Serialization (Functional Core)
- [ ] Create `src/Duet/Rpc/JsonRpc/Framing/Writer.hs` module
- [ ] Implement `buildFramedMessage :: ByteString -> ByteString` (pure)
  - Calculate UTF-8 byte length
  - Build `Content-Length: N\r\n\r\n` header
  - Concatenate header and body
  - Return complete framed message

### 2.4 Implement I/O Operations (Imperative Shell)
- [ ] Create `src/Duet/Rpc/JsonRpc/Framing/IO.hs` module
- [ ] Implement `readFrameHeader :: Handle -> IO (Either FramingError FrameHeader)`
  - Read bytes until `\r\n\r\n` with 8 KB limit
  - Call pure `parseHeaders` function
  - Use timeout (30 seconds) for read operation
  - Return timeout as special case (no error response, log and continue)
- [ ] Implement `readFrameBody :: Handle -> Int -> IO (Either FramingError ByteString)`
  - Call pure `validateContentLength` first
  - Read exactly N bytes with timeout (30 seconds)
  - Call pure `validateBodyLength` after read
  - Return timeout as special case (discard partial, log and continue)
- [ ] Implement `readFrame :: Handle -> IO (Either FramingError ByteString)`
  - Orchestrate: read header, validate, read body, validate
  - Handle timeouts separately (log, reset state, return special marker)
- [ ] Implement `writeFrame :: Handle -> ByteString -> IO ()`
  - Call pure `buildFramedMessage`
  - Write to handle
  - Flush immediately

---

## Phase 3: JSON-RPC Message Handling

### 3.1 Implement Pure Request Parsing (Functional Core)
- [ ] Create `src/Duet/Rpc/JsonRpc/Request.hs` module
- [ ] Implement `parseRequest :: ByteString -> Either JsonRpcError JsonRpcRequest` (pure)
  - Decode JSON from UTF-8 bytes
  - Check if result is JSON array → reject with -32600 "Batch requests not supported" and error.data.reason: "batch-not-supported"
  - Check for required `jsonrpc` field = "2.0"
  - Check for required `method` field (string)
  - Parse optional `params` field
  - Parse optional `id` field (use custom RequestId parser that rejects object/array/boolean)
  - Return appropriate JSON-RPC error for validation failures
- [ ] Implement `isNotification :: JsonRpcRequest -> Bool` (pure)
  - Check if `id` field is absent (not just null - actually missing)
  - Return True if missing, False otherwise

### 3.2 Implement Pure Response Serialization (Functional Core)
- [ ] Create `src/Duet/Rpc/JsonRpc/Response.hs` module
- [ ] Implement `buildSuccessResponse :: RequestId -> Value -> JsonRpcResponse` (pure)
  - Construct response with jsonrpc="2.0", result, and id
  - Ensure `id` is echoed exactly as received (preserving type)
- [ ] Implement `buildErrorResponse :: Maybe RequestId -> Int -> Text -> Maybe Value -> JsonRpcError` (pure)
  - Construct error response with jsonrpc="2.0", error details, and id
  - Use canonical error messages per ADR-012
  - Include minimal error.data per ADR-012
  - Use `id: null` if RequestId is Nothing (unparseable or oversize)
- [ ] Implement `serializeResponse :: JsonRpcResponse -> ByteString` (pure)
  - Encode response to JSON
  - Convert to UTF-8 bytes
- [ ] Implement `serializeError :: JsonRpcError -> ByteString` (pure)
  - Encode error response to JSON
  - Convert to UTF-8 bytes

### 3.3 Implement Error Code Mapping (Functional Core)
- [ ] Create `src/Duet/Rpc/JsonRpc/ErrorCodes.hs` module
- [ ] Define standard JSON-RPC error codes as constants
  - `parseErrorCode = -32700`
  - `invalidRequestCode = -32600`
  - `methodNotFoundCode = -32601`
  - `invalidParamsCode = -32602`
  - `internalErrorCode = -32603`
- [ ] Define canonical error messages per ADR-012
  - Pure mapping functions for each error type

---

## Phase 4: RPC Method Implementations

### 4.1 Define Method Handler Types (Functional Core + Shell Interface)
- [ ] Create `src/Duet/Rpc/JsonRpc/Methods/Types.hs` module
- [ ] Define `MethodContext` type for shared state (pure data):
  - `logEnv :: LogEnv` - for setLogLevel
  - `shutdownVar :: TVar Bool` - for shutdown coordination
  - `serverVersion :: Text` - from Paths_duet_rpc (computed once at startup)
- [ ] Define `MethodHandler` type: `MethodContext -> Value -> IO (Either JsonRpcError Value)`
  - Takes context, params, returns result or error
- [ ] Define `MethodRegistry` type: `Map Text MethodHandler`

### 4.2 Implement Pure Method Logic (Functional Core)
- [ ] Create `src/Duet/Rpc/JsonRpc/Methods/Core.hs` module
- [ ] Implement pure response builders (no IO):
  - `buildInitializeResponse :: Text -> Value` (takes version)
  - `buildListMethodsResponse :: Value` (static list)
  - `buildDescribeMethodsResponse :: Value` (static list)
  - `buildVersionResponse :: Text -> Value` (takes version)
  - `buildShutdownResponse :: Value` (static message)

- [ ] Implement pure validation functions:
  - `validateNoParams :: Value -> Either JsonRpcError ()` (for methods that take no params)
  - `parseLogLevel :: Value -> Either JsonRpcError Text` (extract and normalize level)
  - `validateLogLevel :: Text -> Either JsonRpcError Text` (check against accepted values, return normalized)

### 4.3 Implement Method Handlers (Imperative Shell)
- [ ] Create `src/Duet/Rpc/JsonRpc/Methods/Handlers.hs` module
- [ ] Implement `handleInitialize :: MethodHandler`
  - Call pure `validateNoParams`
  - Call pure `buildInitializeResponse` with version from context
  - Return result (no IO needed beyond validation)

- [ ] Implement `handleListMethods :: MethodHandler`
  - Call pure `validateNoParams`
  - Call pure `buildListMethodsResponse`
  - Return result

- [ ] Implement `handleDescribeMethods :: MethodHandler`
  - Call pure `validateNoParams`
  - Call pure `buildDescribeMethodsResponse`
  - Return result

- [ ] Implement `handleVersion :: MethodHandler`
  - Call pure `validateNoParams`
  - Call pure `buildVersionResponse` with version from context
  - Return result

- [ ] Implement `handleSetLogLevel :: MethodHandler`
  - Call pure `parseLogLevel` to extract level from params
  - Call pure `validateLogLevel` to validate and normalize
  - Perform IO: update logger severity using Katip runtime configuration
  - Build and return success response: `{"level": "normalized", "success": true}`

- [ ] Implement `handleShutdown :: MethodHandler`
  - Call pure `validateNoParams`
  - Perform IO: set shutdown flag in TVar using STM
  - Call pure `buildShutdownResponse`
  - Return result (shutdown sequence happens in main loop)

### 4.4 Build Method Registry (Imperative Shell)
- [ ] Create `src/Duet/Rpc/JsonRpc/Methods/Registry.hs` module
- [ ] Implement `buildMethodRegistry :: MethodRegistry`
  - Map method names to handlers:
    - "initialize" → handleInitialize
    - "listMethods" → handleListMethods
    - "describeMethods" → handleDescribeMethods
    - "version" → handleVersion
    - "setLogLevel" → handleSetLogLevel
    - "shutdown" → handleShutdown
- [ ] Implement `lookupMethod :: Text -> MethodRegistry -> Maybe MethodHandler` (pure)
  - Simple Map.lookup wrapper

---

## Phase 5: Request Dispatcher

### 5.1 Define Dispatcher Types (Functional Core)
- [ ] Create `src/Duet/Rpc/JsonRpc/Dispatcher/Types.hs` module
- [ ] Define `DispatchResult` ADT:
  - `DispatchSuccess JsonRpcResponse` - method executed successfully
  - `DispatchError JsonRpcError` - method failed with error
  - `DispatchNotification` - notification processed (no response)
  - `DispatchShutdown (Maybe JsonRpcResponse)` - shutdown triggered (optional response if request, not notification)
- [ ] Define `DispatchContext` type:
  - `methodRegistry :: MethodRegistry`
  - `methodContext :: MethodContext`
  - `shutdownVar :: TVar Bool`

### 5.2 Implement Pure Dispatch Logic (Functional Core)
- [ ] Create `src/Duet/Rpc/JsonRpc/Dispatcher/Core.hs` module
- [ ] Implement `routeRequest :: JsonRpcRequest -> MethodRegistry -> Either JsonRpcError MethodHandler` (pure)
  - Look up method name in registry
  - Return handler if found
  - Return -32601 "Method not found" error if not found
- [ ] Implement `buildErrorFromException :: Exception e => e -> Maybe RequestId -> JsonRpcError` (pure)
  - Map exceptions to -32603 "Internal error"
  - Use safe error message (no stack trace, paths, or secrets)
  - Include request ID if available

### 5.3 Implement Dispatch Orchestration (Imperative Shell)
- [ ] Create `src/Duet/Rpc/JsonRpc/Dispatcher/Loop.hs` module
- [ ] Implement `dispatchRequest :: DispatchContext -> JsonRpcRequest -> IO DispatchResult`
  - Check if shutdown already triggered (read TVar) → return early if yes
  - Check if notification using pure `isNotification` → set flag
  - Call pure `routeRequest` to find handler
  - If method not found:
    - Log at `warn` level with method name and id
    - Return DispatchError if request, DispatchNotification if notification
  - If handler found:
    - Execute handler (catch exceptions, map to -32603)
    - Log exceptions at `error` level
    - Check if shutdown method → return DispatchShutdown
    - Return DispatchSuccess or DispatchError based on handler result
    - Return DispatchNotification if notification (no response)

- [ ] Implement `processDispatchResult :: DispatchResult -> IO (Maybe ByteString)`
  - DispatchSuccess → serialize response, return Just bytes
  - DispatchError → serialize error, return Just bytes
  - DispatchNotification → return Nothing (no response)
  - DispatchShutdown → serialize response if present, return Just/Nothing

- [ ] Implement `runDispatchLoop :: DispatchContext -> Handle -> Handle -> IO ()`
  - Single-threaded sequential loop per ADR-011
  - Read one framed message from stdin using Framing.IO.readFrame
  - Handle framing errors (timeout → log and continue, others → send error response)
  - Parse JSON-RPC request using pure Request.parseRequest
  - Dispatch request using `dispatchRequest`
  - Process result using `processDispatchResult`
  - Write response if present using Framing.IO.writeFrame
  - Check shutdown flag, exit gracefully if set
  - Repeat until EOF or shutdown
  - EOF triggers graceful shutdown (logged at info)

### 5.4 Error Handling Strategy (Imperative Shell)
- [ ] Implement logging hooks in dispatch loop:
  - Framing errors → log at `warn` with error details
  - Parse errors → log at `warn` with method/id when available
  - Method not found → log at `warn` with method name and id
  - Invalid params → log at `warn` with method name and id
  - Handler exceptions → log at `error` with method name, id, and safe error message
  - Unknown-method notifications → log at `warn` with method name
- [ ] Ensure daemon stays alive for all client errors (-32700, -32600, -32601, -32602)
- [ ] Ensure daemon stays alive for internal errors (-32603)
- [ ] Only exit for: shutdown request/notification, EOF, unrecoverable IO errors

### 5.5 Response Ordering Guarantee (Natural from Design)
- [ ] Verify single-threaded execution preserves order
  - Process request completely before reading next
  - Write response before reading next request
  - No concurrent handlers or queuing in v1

---

## Phase 6: Daemon Lifecycle & Signals

### 6.1 Define Lifecycle Types (Functional Core)
- [ ] Create `src/Duet/Rpc/JsonRpc/Lifecycle/Types.hs` module
- [ ] Define `ShutdownTrigger` ADT:
  - `ShutdownByRequest` - shutdown RPC method called
  - `ShutdownByEOF` - stdin closed
  - `ShutdownBySignal Text` - signal received (name of signal)
- [ ] Define `ShutdownState` type:
  - `shutdownTriggered :: Bool`
  - `trigger :: Maybe ShutdownTrigger`
  - Use STM TVar for concurrent access

### 6.2 Implement Signal Handlers (Imperative Shell)
- [ ] Create `src/Duet/Rpc/JsonRpc/Lifecycle/Signals.hs` module
- [ ] Implement `installSignalHandlers :: TVar ShutdownState -> LogEnv -> IO ()`
  - Install handlers for SIGINT, SIGTERM, SIGHUP
  - On signal: atomically set shutdown flag with trigger type
  - Log signal name at `info` level when received
  - Use System.Posix.Signals.installHandler
  - All signals trigger same graceful shutdown behavior

### 6.3 Implement Shutdown Coordination (Imperative Shell)
- [ ] Create `src/Duet/Rpc/JsonRpc/Lifecycle/Shutdown.hs` module
- [ ] Implement `initiateShutdown :: TVar ShutdownState -> ShutdownTrigger -> STM ()`
  - Atomically set shutdown flag and trigger in TVar
  - Pure STM transaction (no IO)

- [ ] Implement `checkShutdown :: TVar ShutdownState -> STM Bool`
  - Read shutdown flag from TVar
  - Pure STM transaction

- [ ] Implement `gracefulShutdown :: LogEnv -> ShutdownTrigger -> IO ()`
  - Log shutdown trigger at `info` level:
    - ShutdownByEOF → "stdin closed, shutting down gracefully"
    - ShutdownBySignal name → "received {name}, shutting down gracefully"
    - ShutdownByRequest → "shutdown requested, shutting down gracefully"
  - Flush stdout
  - Flush and close Katip log scribes with timeout
  - Return within 2-second total deadline

### 6.4 Implement Cleanup Operations (Imperative Shell)
- [ ] Implement `flushWithTimeout :: Handle -> Int -> IO Bool`
  - Attempt to flush handle
  - Use timeout (milliseconds) with System.Timeout
  - Return True if successful, False if timeout
  - Cap at specified timeout (500ms for logs)

- [ ] Implement `cleanupResources :: LogEnv -> IO ()`
  - Flush stdout (no timeout, must complete)
  - Flush logs with 500ms timeout using `flushWithTimeout`
  - Close Katip scribes
  - Exit anyway if log flush times out (to meet 2-second deadline)

### 6.5 Implement Exit Code Strategy (Imperative Shell)
- [ ] Create `src/Duet/Rpc/JsonRpc/Lifecycle/Exit.hs` module
- [ ] Define exit codes as constants:
  - `exitGraceful = 0` - normal shutdown
  - `exitUnrecoverable = 1` - fatal error
- [ ] Implement `exitWithCode :: Int -> IO a`
  - Call System.Exit.exitWith
  - Wrapper for clarity in main

### 6.6 Implement Graceful Shutdown Deadline (Imperative Shell)
- [ ] Implement `withShutdownDeadline :: IO () -> IO ()`
  - Wrap shutdown sequence in 2-second timeout
  - If current request is `shutdown`, reply then exit
  - If other request running, try to finish and send response
  - Exit without response if not done within 2 seconds
  - Use async timeout mechanism

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
2. **Framing Layer**: I/O boundary that everything depends on (pure parsing + impure I/O)
3. **JSON-RPC Messages**: Protocol layer built on framing (pure transformations)
4. **Methods**: Business logic requires protocol support (pure logic + minimal I/O handlers)
5. **Dispatcher**: Orchestrates methods and protocol (imperative shell calling pure core)
6. **Lifecycle**: Requires dispatcher and methods in place (imperative coordination)
7. **Logging**: Cross-cutting concern integrated throughout (impure but isolated)
8. **CLI Integration**: Wires everything together (top-level imperative shell)
9. **Testing**: Validates each layer and integration (pure functions trivial to test)
10. **Documentation**: Final polish and validation

### Key Principles
- **KISS**: Implement exactly what's in the story, no extras
- **YAGNI**: Don't build for future requirements (e.g., no concurrent dispatcher, no configurable limits)
- **Imperative Shell, Functional Core**: Maximize pure functions, push I/O to edges
- **Fail Gracefully**: Daemon stays alive for client errors, logs and reports properly
- **Testability**: Pure functions test without I/O, integration tests validate shell

### Imperative Shell, Functional Core Architecture

**Functional Core (Pure)**:
- All parsing: headers, JSON-RPC requests, validation
- All building: responses, errors, framed messages
- All mapping: errors to codes, triggers to messages
- All routing: method lookup, dispatch decisions
- Type: `a -> Either Error b` or `a -> b`
- Testing: Unit tests with pure inputs/outputs

**Imperative Shell (Impure)**:
- I/O operations: reading stdin, writing stdout/stderr
- Effects: logging, signal handling, STM coordination
- Orchestration: dispatch loop, lifecycle management
- Type: includes `IO` monad
- Testing: Integration tests with real I/O or mocks

**Benefits of This Approach**:
1. **Testability**: Pure functions trivial to test (no mocking, no IO setup)
2. **Reasoning**: Pure core has no hidden state or effects
3. **Concurrency**: Pure functions inherently thread-safe
4. **Composition**: Pure functions compose naturally
5. **Edge Complexity**: All complexity (signals, timeouts, I/O) isolated at edges

**Example Structure**:
```
Framing/
  Types.hs       -- Pure types and error ADTs
  Parser.hs      -- Pure: ByteString -> Either Error Header
  Writer.hs      -- Pure: Message -> ByteString
  IO.hs          -- Impure: Handle -> IO (Either Error ByteString)
```

### Critical Requirements
- **Stdout discipline**: ONLY framed JSON-RPC responses on stdout (enforced in shell)
- **Sequential processing**: No concurrent requests in v1 (natural from single-threaded shell)
- **Graceful shutdown**: Always within 2 seconds (timeout in shell)
- **Error data**: Minimal, non-sensitive, per ADR-012 (built by pure functions)
- **Logging**: Structured, redacted, never to stdout (shell responsibility)
- **Pure core**: All business logic testable without I/O (functional core)
- **Thin shell**: Minimal orchestration logic, delegates to pure functions
