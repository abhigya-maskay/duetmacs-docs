# Story 003 JSON-RPC Daemon - Risk Register

## Scope Note
This risk register is strictly scoped to Story 003 (JSON-RPC Daemon Interface).

## Risk Register

| Title | Story | Category | Description | Likelihood | Impact | Severity | Status |
|---|---|---|---|---|---|---|---|
| Emacs jsonrpc.el compatibility gap | 003 | Integrations | LSP-style framing compatibility assumed but not validated until Story 004; protocol divergence could block Emacs integration | High | High | Critical | Mitigation Planned |
| Log file unbounded growth | 003 | Performance | DUET_RPC_LOG lacks rotation/size limits; long-running daemons could fill disk | Medium | High | Critical | Mitigation Planned |
| No protocol versioning strategy | 003 | Legacy | No mechanism to evolve RPC methods or deprecate old ones; breaking changes require full client rewrites | High | Medium | High | Mitigation Planned |
| Katip logging library failure | 003 | Integrations | External dependency on Katip; library bugs or compatibility issues could break logging infrastructure | Low | High | Medium | Risk Accepted |
| Single-threaded processing bottleneck | 003 | Performance | Sequential request handling; slow methods block all subsequent requests indefinitely | Medium | Medium | Medium | Risk Accepted |
| Log file permission exposure | 003 | Security | DUET_RPC_LOG file permissions unspecified; could create world-readable logs containing debug data | Medium | Medium | Medium | Mitigation Planned |
| In-flight request loss on shutdown | 003 | Performance | 2-second shutdown window may truncate responses for long-running requests; clients receive no response | Medium | Medium | Medium | Risk Accepted |
| stdin EOF no reconnect path | 003 | Integrations | stdin EOF triggers permanent shutdown; no mechanism to restart or reconnect without process restart | Low | Medium | Low | Risk Accepted |

## Risk Acceptance Decisions

### Katip Logging Library Failure (Integration Risk)
**Decision**: Risk Accepted
**Date**: 2025-10-03
**Rationale**: Katip is mature, well-maintained library in Haskell ecosystem. Low likelihood of catastrophic failure. Logging is non-critical path (daemon continues on log errors). Mitigation effort (custom logging abstraction) outweighs benefit for V1 scope.

### Single-threaded Processing Bottleneck (Performance Risk)
**Decision**: Risk Accepted
**Date**: 2025-10-03
**Rationale**: Story 003 explicitly scopes out concurrent request handling and session state management (lines 21, 97). ADR-011 documents this as "high confidence" assumption that single-threaded handling is sufficient for V1. Concurrency complexity deferred until actual performance issues observed in Emacs client (Story 004).

### In-flight Request Loss on Shutdown (Performance Risk)
**Decision**: Risk Accepted
**Date**: 2025-10-03
**Rationale**: 2-second shutdown window documented in lifecycle requirements (lines 118-124). Story 003 scopes out long-running AI streaming responses (line 18). All V1 methods (initialize, version, setLogLevel, etc.) complete in milliseconds. Risk materializes only in future stories with AI operations - acceptable for foundation story.

### stdin EOF No Reconnect Path (Integration Risk)
**Decision**: Risk Accepted
**Date**: 2025-10-03
**Rationale**: Daemon design follows single-session lifecycle per ADR-013. Emacs client owns process lifecycle and can restart daemon trivially. Reconnect logic adds complexity without clear V1 use case. Deferred pending actual operational need.

## Risk Mitigations

### 1. Emacs jsonrpc.el Compatibility Gap (Critical)
**Evidence**: Story 003 claims LSP-style framing compatibility with jsonrpc.el (line 80) but provides no validation
**Mitigation Status**: PLANNED - Integration test harness designed
**Mitigation Details**:

#### Mitigation 1: Emacs Integration Test Suite
Create functional test that validates protocol compatibility using actual Emacs jsonrpc.el client:

```elisp
;; test/integration/emacs-client-test.el
(require 'jsonrpc)

(defun duet-rpc-integration-test ()
  "Test duet-rpc daemon compatibility with jsonrpc.el"
  (let* ((proc (make-process
                :name "duet-rpc-test"
                :command '("duet-rpc" "rpc")
                :connection-type 'pipe))
         (client (jsonrpc-process-connection
                  :name "duet-rpc-client"
                  :process proc)))

    ;; Test 1: initialize method
    (let ((response (jsonrpc-request client 'initialize nil)))
      (should (string= (alist-get 'name (alist-get 'serverInfo response))
                       "duet-rpc"))
      (should (string= (alist-get 'protocolVersion response) "2.0")))

    ;; Test 2: version method
    (let ((response (jsonrpc-request client 'version nil)))
      (should (string-match "^[0-9]+\\.[0-9]+\\.[0-9]+$"
                           (alist-get 'version response))))

    ;; Test 3: shutdown
    (jsonrpc-request client 'shutdown nil)
    (sleep-for 0.5)
    (should (eq (process-status proc) 'exit))
    (should (= (process-exit-status proc) 0))))
```

**Execution**: Run in CI via emacs --batch --eval during test phase
**Impact**: Validates framing protocol before Story 004 Emacs client work begins
**Success Criteria**: All 6 RPC methods callable from jsonrpc.el with correct responses

#### Mitigation 2: Protocol Compliance Test Cases
Extend functional test suite to validate LSP framing edge cases:

```haskell
-- test/DuetRpc/Protocol/FramingSpec.hs
spec :: Spec
spec = describe "LSP Framing Compatibility" $ do

  it "handles Content-Type with charset variations" $
    forM_ ["utf-8", "UTF-8", "Utf-8"] $ \charset -> do
      let headers = "Content-Type: application/vscode-jsonrpc; charset=" <> charset
      parseContentType headers `shouldBe` Right ApplicationJsonRpc

  it "accepts framed messages without Content-Type header" $ do
    let frame = "Content-Length: 45\r\n\r\n{\"jsonrpc\":\"2.0\",\"method\":\"version\",\"id\":1}"
    parseFrame frame `shouldSatisfy` isRight

  it "handles back-to-back framed messages" $ do
    let frame1 = "Content-Length: 45\r\n\r\n{\"jsonrpc\":\"2.0\",\"method\":\"version\",\"id\":1}"
        frame2 = "Content-Length: 50\r\n\r\n{\"jsonrpc\":\"2.0\",\"method\":\"initialize\",\"id\":2}"
    parseFrames (frame1 <> frame2) `shouldReturn` [msg1, msg2]
```

**Impact**: Prevents regressions in framing implementation
**Coverage**: All acceptance criteria in lines 162-172

### 2. Log File Unbounded Growth (Critical)
**Evidence**: DUET_RPC_LOG lacks size limits or rotation (line 103); long-running daemons could exhaust disk
**Mitigation Status**: PLANNED - Size cap and rotation designed
**Mitigation Details**:

#### Mitigation 1: Log File Size Cap with Rotation
Implement automatic log rotation when file size exceeds threshold:

```haskell
-- src/DuetRpc/Logging.hs
data LogConfig = LogConfig
  { logSink :: LogSink
  , logLevel :: Severity
  , logMaxSize :: Int64  -- bytes, default 10 MB
  , logMaxFiles :: Int   -- rotation count, default 3
  }

setupLogging :: LogConfig -> IO LogEnv
setupLogging cfg = do
  case logSink cfg of
    LogFile path -> do
      -- Check current size before writing
      size <- getFileSize path `catch` \(_ :: IOException) -> pure 0
      when (size >= logMaxSize cfg) $ rotateLogFile path (logMaxFiles cfg)
      -- Use katip's FileScribe with size monitoring
      scribe <- mkFileScribe path (permitItem (logLevel cfg)) V3
      -- ... rest of setup
```

**Rotation Strategy**:
- `duet-rpc.log` → `duet-rpc.log.1` → `duet-rpc.log.2` → deleted
- Default: 10 MB per file × 3 files = 30 MB max disk usage
- Configurable via env var `DUET_RPC_LOG_MAX_SIZE` (bytes)

**Fallback**: If rotation fails (permissions, disk full), fall back to stderr with warning

**Impact**: Bounds disk usage to predictable limit regardless of daemon uptime
**Testing**: Functional test generates >10 MB logs and verifies rotation

#### Mitigation 2: Startup Disk Space Check
Validate available disk space before starting daemon:

```haskell
-- src/DuetRpc/Daemon.hs
startDaemon :: Config -> IO ()
startDaemon cfg = do
  when (isFileLogging cfg) $ do
    let logPath = getLogPath cfg
        logDir = takeDirectory logPath
    available <- getAvailableSpace logDir
    let required = logMaxSize cfg * fromIntegral (logMaxFiles cfg)
    when (available < required * 2) $  -- 2x safety margin
      logWarn $ "Low disk space: " <> show available
                <> " bytes available, " <> show required <> " required"
  -- ... rest of startup
```

**Impact**: Early warning for disk space issues
**Threshold**: 2× max log size (60 MB default) to account for other operations

### 3. No Protocol Versioning Strategy (High)
**Evidence**: No version negotiation in initialize method; no deprecation path for method changes
**Mitigation Status**: PLANNED - Protocol version negotiation designed
**Mitigation Details**:

#### Mitigation 1: Enhanced Initialize Handshake
Add protocol version negotiation to initialize method:

```haskell
-- Response from initialize method
{ "serverInfo":
    { "name": "duet-rpc"
    , "version": "0.1.0"
    }
, "protocolVersion": "2.0"
, "capabilities":
    { "supportedProtocolVersions": ["2.0"]
    , "supportedFeatures":
        { "notifications": true
        , "batchRequests": false
        , "streaming": false  -- future: AI streaming responses
        }
    }
}
```

**Client Behavior**:
- Clients should call `initialize` first and check `supportedProtocolVersions`
- Server rejects requests from clients announcing incompatible protocol version
- Future breaking changes increment protocol version (e.g., "3.0")

**Impact**: Enables protocol evolution without breaking existing clients

#### Mitigation 2: Method Deprecation Response Header
Add deprecation warnings to method responses:

```haskell
-- Example: Deprecating old method signature
{ "jsonrpc": "2.0"
, "id": 1
, "result": { "level": "debug", "success": true }
, "deprecated":
    { "message": "setLogLevel params changed in v2.0; use {level: string, persist: bool}"
    , "sunset": "2026-01-01"  -- ISO 8601 date when support ends
    , "replacement": "setLogLevel2"
    }
}
```

**Policy**:
- Deprecated methods supported for 6 months minimum
- Deprecation logged at `warn` level on server
- Clients should migrate before sunset date

**Impact**: Graceful migration path for breaking changes

#### Mitigation 3: Method Versioning Convention
Establish naming convention for method evolution:

- **Non-breaking changes**: Same method name (e.g., add optional param)
- **Breaking changes**: New method with version suffix (e.g., `setLogLevel` → `setLogLevel2`)
- **Major protocol bump**: Incompatible clients rejected at initialize

**Documentation**: Capture in new ADR-016 (Protocol Versioning Strategy)

### 4. Log File Permission Exposure (Medium)
**Evidence**: DUET_RPC_LOG file creation lacks explicit permission controls (line 103)
**Mitigation Status**: PLANNED - Restrictive permissions on creation
**Mitigation Details**:

#### Mitigation: Secure Log File Creation
Create log file with restrictive permissions (0600 = owner read/write only):

```haskell
-- src/DuetRpc/Logging.hs
import System.Posix.Files (setFileMode, ownerReadMode, ownerWriteMode)
import System.IO (openFile, WriteMode, hClose)

createSecureLogFile :: FilePath -> IO ()
createSecureLogFile path = do
  -- Create file with default permissions
  handle <- openFile path WriteMode
  hClose handle

  -- Immediately restrict to owner-only
  let mode = ownerReadMode `unionFileModes` ownerWriteMode  -- 0600
  setFileMode path mode

setupFileLogging :: FilePath -> Severity -> IO Scribe
setupFileLogging path level = do
  -- Ensure secure permissions before writing sensitive data
  createSecureLogFile path
  mkFileScribe path (permitItem level) V3
```

**Permissions**:
- Log file: `0600` (owner read/write only)
- Parent directory: Inherits umask (user responsible for securing `~/.config/duet-rpc/` etc.)

**Platform Support**:
- Unix/Linux/macOS: Full support via `System.Posix.Files`
- Windows: Best effort via `System.Directory.setPermissions` (ACLs differ)

**Impact**: Prevents unauthorized access to log files containing debug data
**Testing**: Functional test verifies file mode after creation

**Documentation**: Update line 103 with permission requirements

---

## Risk Categories Coverage

- **Performance**: 3 risks identified (log growth, single-threaded, shutdown truncation)
- **Security**: 1 risk identified (log file permissions)
- **Integrations**: 3 risks identified (jsonrpc.el compatibility, Katip dependency, stdin EOF)
- **Legacy**: 1 risk identified (protocol versioning)
