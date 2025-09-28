---
tags: [testing, test-strategy, cli, story-001]
aliases: [Story 001 Tests, CLI Bootstrap Tests]
---

# Test Strategy — Story 001: CLI Version/Help Bootstrap

## Scope Boundary
- **Parent Story**: [[001-cli-version-help-bootstrap|Story 001]]
- **Strict Focus**: Testing ONLY version/help/bootstrap functionality
- **Must NOT test**: Features from other stories (doctor, rpc, prompt)

## Objectives
- Verify `duet-rpc` installs and responds with version/help consistently.
- Establish deterministic behavior for logging, color, and output.
- Anchor future stories with a stable, testable CLI entrypoint.

## In Scope
- Commands: `--version`, `version`, `--help`, no-args help.
- Global options: `-V/--version`, `-h/--help`, `--log-level`, `--no-color`.
- Environment variables: `DUET_RPC_LOG`, `NO_COLOR`.
- Error handling for unknown subcommands/flags.
- Output format: newline-terminated, color rules, help structure.
 

## Out of Scope
- Behavior for doctor, rpc, prompt commands (separate stories)
- Full configuration loading/validation (skeleton only)
- Packaging, release, and editor integration
- Cross-story integration tests
 

## Inputs & References
- Story: [[001-cli-version-help-bootstrap|Story 001: CLI Version/Help Bootstrap]]
- Components: See [[05-Components/CLI-Components/README|CLI Components]]

## Approach
- Feature-first, risk-based coverage with P0 critical-path priority.
- Start with E2E command invocations; add integration/unit for determinism.
- Use golden snapshots for help text; regex-pattern assertions for logs and ANSI detection.
- Control environment (env vars, TTY vs pipe) to validate modes.

## Test Levels & Coverage Targets
- Unit: 70–80% on changed code; 100% on critical branches (VersionManager, ErrorHandler, CLI parser critical paths).
- Integration: 100% of P0 interfaces (CLI routing, help generation via optparse-applicative).
- E2E: 100% of P0 flows (version/help, error on unknown).
- Optional: Property tests (hedgehog) on parser invariants where valuable.

## Prioritization
- P0: Version/help correctness; error handling; color rules off paths; logging debug path; newline termination.
- P1: Short flags parity; color-on-TTY; log redirection to file; default log-level behavior.
- P2: Config skeleton constants existence and precedence doc.

## Test Matrix (Selected)

### Version
- E2E | P0 | `duet-rpc --version` prints semver.
- E2E | P0 | `duet-rpc version` prints same semver.
- Unit | P0 | VersionManager returns single-source version; formatted consistently.

### Help
- E2E | P0 | `duet-rpc --help` and `duet-rpc` show synopsis, subcommands, footer.
- Integration | P0 | Unknown subcommand/flag → error + usage; no stack trace.
- Unit | P1 | HelpFormatter/CLI metadata builds synopsis `duet-rpc [COMMAND] [OPTIONS]`, lists `version`, `doctor`, `rpc`, `prompt`, footer.

### Color/TTY
- E2E | P0 | `NO_COLOR=1` or `--no-color` → no ANSI in help.
- E2E | P1 | TTY attached → colors present; piped → plain.

### Logging
- E2E | P0 | `--log-level debug version` emits structured debug logs to stderr; stdout is version.
- E2E | P1 | Default warn+ only; `version` outputs no logs by default.
- E2E | P1 | `DUET_RPC_LOG=<file>` routes logs to file; stderr quiet.
- E2E | P1 | `DUET_RPC_LOG` invalid path → logger falls back to stderr with a warning; command output/exit unchanged.
- **Format Guard**: `test/Duet/Rpc/Test/CLI/Logging.hs` validates that structured log lines start with Katip’s bracketed prefix `[timestamp][namespace][Level]` (timestamp accepted in ISO-8601 or local form) before message text, keeping observability docs and tests in sync.

### Format
- E2E | P0 | `--version`/`--help` complete ~100ms (CI threshold relaxed).
- E2E | P0 | All outputs newline-terminated.

## Key Test Cases
- T-CLI-VER-001 (P0): `duet-rpc --version`
  - Expect: stdout `<semver>\n`; stderr empty.
- T-CLI-VER-002 (P0): `duet-rpc version`
  - Expect: same as T-CLI-VER-001; versions equal.
- T-CLI-HLP-001 (P0): `duet-rpc --help`
  - Expect: synopsis `duet-rpc [COMMAND] [OPTIONS]`; lists `version`, `doctor`, `rpc`, `prompt`; footer “See 'duet-rpc <command> --help'...”; newline.
- T-CLI-HLP-002 (P0): `duet-rpc` (no args)
  - Expect: same as `--help`.
- T-CLI-ERR-001 (P0): `duet-rpc frobnicate`
  - Expect: error + usage on stderr; no stack trace; stdout empty.
- T-CLI-ERR-002 (P0): `duet-rpc --frobnicate`
  - Expect: same stderr usage text; stdout empty; exit `1`; no stack trace.
- T-CLI-CLR-001 (P0): `NO_COLOR=1 duet-rpc --help`
  - Expect: no ANSI sequences.
- T-CLI-CLR-002 (P0): `duet-rpc --no-color --help`
  - Expect: no ANSI regardless of TTY.
- T-CLI-TTY-001 (P1): TTY-attached `duet-rpc --help`
  - Expect: ANSI color present; piped variant plain.
- T-CLI-LOG-001 (P0): `duet-rpc --log-level debug version`
  - Expect: stderr structured log line(s) with timestamp/level/message; stdout version.
- T-CLI-LOG-002 (P1): `DUET_RPC_LOG=$TMP/duet.log duet-rpc version`
  - Expect: logs written to file; stderr quiet; file lines include timestamp/level/message.
- T-CLI-LOG-NEG-001 (P1): `DUET_RPC_LOG=$TMP/nonexistent/subdir/file.log duet-rpc --log-level debug version`
  - Expect: stderr contains a clear warning about failing to open/write log file (no stack trace); stdout version.
 
- T-CLI-SYN-001 (P1): `-h` and `-V` parity with long flags.
- T-CLI-FMT-001 (P0): Newline at end of stdout/stderr outputs.

## Data & Environment Strategy
- Process: Spawn `duet-rpc`; capture stdout and stderr separately.
- Env control: Set `NO_COLOR`, `DUET_RPC_LOG`, `--log-level` per test; isolate with per-test temp dirs.
- TTY simulation: Prefer `script -q -c "<cmd>" /dev/null` on Unix-like CI to emulate a TTY; gate or use ConPTY alternatives on Windows.
- Assertions: Golden help text (plain variant) via tasty-golden; regex for ANSI absence/presence and log structure; compare `version` outputs for equality.
 
- Determinism: Avoid asserting timestamps; match ISO-8601 pattern and level names only.
  - Helper `assertStructuredLogPrefix` parses the first three Katip brackets and tolerates ISO-8601 (`2025-09-28T17:45:12Z`) or local (`2025-09-28 17:45:12`) timestamps while ensuring the `[Level]` token stays capitalized.
  - For invalid log path, assert presence of a warning substring (e.g., "log" and "failed"/"cannot") rather than exact wording.

## Traceability (AC → Tests)
- `--version` prints semver → T-CLI-VER-001
- `version` prints same semver → T-CLI-VER-002
- `--help` shows synopsis/subcommands/footer → T-CLI-HLP-001
- No args shows help → T-CLI-HLP-002
- Help lists `doctor`, `rpc`, `prompt` → T-CLI-HLP-001 golden
- Unknown subcommand → error + usage; no stack trace → T-CLI-ERR-001
- Unknown flag → error + usage; no stack trace → T-CLI-ERR-002
- Newline-terminated output → T-CLI-FMT-001
 
- `--log-level debug` logs structured debug → T-CLI-LOG-001
- Default warn+ only → T-CLI-LOG-001 (stderr quiet on success)
- `DUET_RPC_LOG` to file → T-CLI-LOG-002
- Invalid `DUET_RPC_LOG` path → fallback warning to stderr; command unaffected → T-CLI-LOG-NEG-001
- `NO_COLOR` or `--no-color` → plain → T-CLI-CLR-001/002
- TTY without NO_COLOR → colored help → T-CLI-TTY-001

## Tooling (Haskell Stack)
- Runner: `tasty` with `tasty-hunit` for assertions; `tasty-hedgehog` for properties where useful.
- Golden snapshots: `tasty-golden` to store help text (plain variant) under `test/golden/`.
- Process spawning: `typed-process` or `System.Process` to run the built binary; ensure `cabal build` artifacts are discoverable in tests.
- Temp files/dirs: `temporary` for per-test isolation (e.g., log file paths).
- ANSI checks: Regex for `\x1b\[[0-9;]*m` to detect color sequences.
- TTY emulation: Use `script -q -c` on Unix to force TTY; conditionally skip on Windows or use platform-appropriate PTY.
- Commands: `cabal test` to run; `cabal run duet-rpc -- --help` for manual smoke.

## Entry/Exit Criteria
- Entry: `duet-rpc` builds and is invocable in test environment.
- Exit: All P0 pass; no open severity-1 defects; golden baselines approved.

## Risks & Mitigations
- PTY flakiness on Windows → gate or use ConPTY; ensure non-TTY path covered everywhere.
 
- Help text drift → review golden diffs; update intentionally.

## Open Questions
- Decision: For Story 001, tests cover `--log-level` only; a `--debug` alias (if added) will be covered in a later story.
- Decision: CI OS matrix — Linux/macOS run full suite including TTY-based color tests (via `script`); Windows skips TTY color tests and runs non-TTY and NO_COLOR/`--no-color` tests.
- Decision: Include negative case for invalid `DUET_RPC_LOG` path — logger must fall back to stderr with a warning.
