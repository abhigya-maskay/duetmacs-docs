---
tags: [story/001, implementation, checklist, cli]
aliases: [Implementation Checklist, Story 001 Implementation Checklist]
---
# Implementation Checklist — Story 001: CLI Version/Help Bootstrap

Use this step-by-step checklist to implement Story 001. After each code step, run the associated test(s).

- [x] 0. Confirm stack and version authority
  - Code: Decide Haskell + Cabal; CLI via `optparse-applicative`; color via `ansi-terminal`; logging via `katip`; version from `Paths_duet_rpc.version`.
  - Test: None (decision checkpoint).

- [x] 1. Scaffold Cabal project and executable
  - Code: Create package with exe `duet-rpc` at version `0.1.0`. Add deps: `optparse-applicative`, `text`, `ansi-terminal`, `katip`, `time`, `directory`, `filepath`. Test deps: `tasty`, `tasty-hunit`, `tasty-golden`, `typed-process`, `temporary`.
  - Test: `cabal build` succeeds. `cabal run duet-rpc -- --help` prints default help.

- [x] 2. Add entrypoint and base CLI wiring
  - Code: Implement `app/Main.hs` with `optparse-applicative`. Enable `showHelpOnEmpty` and `showHelpOnError`. Parse globals: `-V/--version`, `-h/--help`, `--log-level {debug|info|warn|error}`, `--no-color`. Register subcommands: `version`, `doctor`, `rpc`, `prompt` (only `version` wired for now). No args → help. Unknown subcmd/flag → error + usage.
  - Test:
    - T-CLI-HLP-002: `duet-rpc` (no args) shows help.
    - T-CLI-ERR-001: `duet-rpc frobnicate` → error + usage on stderr; stdout empty; no stack trace.

- [x] 3. Implement VersionManager and `version` command
  - Code: Create module to read `Paths_duet_rpc.version` and render semver. Wire `version` subcommand and global `--version` to print the same string to stdout with trailing newline.
  - Test:
    - T-CLI-VER-001: `duet-rpc --version` prints semver; stderr empty.
    - T-CLI-VER-002: `duet-rpc version` equals `--version`.

- [x] 4. Implement OutputFormatter (color/TTY/no-color rules)
  - Code: Detect TTY for stdout/stderr; precedence `--no-color` > `NO_COLOR` > isTTY. Use `ansi-terminal` SGR when enabled. All writes newline-terminated.
  - Test:
    - T-CLI-CLR-001: `NO_COLOR=1 duet-rpc --help` → no ANSI sequences.
    - T-CLI-CLR-002: `duet-rpc --no-color --help` → no ANSI regardless of TTY.

- [x] 5. Shape final help text
  - Code: Configure help: synopsis `duet-rpc [COMMAND] [OPTIONS]`; subcommands `version`, `doctor`, `rpc`, `prompt` with one-liners; footer “See 'duet-rpc <command> --help' for more information.” Use `optparse-applicative` defaults for structure.
  - Test:
    - T-CLI-HLP-001: `duet-rpc --help` (plain mode) matches golden snapshot.
    - T-CLI-HLP-002: `duet-rpc` (no args) equals `--help`.

- [x] 6. Implement Logger with level, stderr/file routing, fallback
  - Code: Initialize `katip` at startup; default level `warn` to stderr. Parse `--log-level {debug|info|warn|error}`. If `DUET_RPC_LOG` set, write logs to file; on invalid path, fall back to stderr with a single clear warning (no stack trace); command output unaffected.
  - Test:
    - T-CLI-LOG-001: `duet-rpc --log-level debug version` → stdout version; stderr has structured debug log(s) with timestamp/level/message.
    - T-CLI-LOG-002 (P1): `DUET_RPC_LOG=$TMP/duet.log duet-rpc version` → logs to file; stderr quiet.
    - T-CLI-LOG-NEG-001 (P1): `DUET_RPC_LOG=$TMP/nonexistent/x/y.log duet-rpc --log-level debug version` → stderr shows warning; stdout version.

  - Test: Re-validate T-CLI-ERR-001 (error + usage, no stack trace).

- [x] 8. Add Config skeleton (constants only)
  - Code: Define config search constants and placeholder config struct. Document precedence: `DUET_RPC_CONFIG` → project `.duet-rpc.toml` → XDG/home. Do not load yet.
  - Test: Unit test asserts constants/types exist and precedence order is documented (P2).

- [x] 9. Ensure newline-terminated outputs everywhere
  - Code: Audit all writes (version, help, errors) to ensure trailing newline.
  - Test: T-CLI-FMT-001: Assert newline at end of stdout/stderr outputs.

- [x] 10. Performance sanity
  - Code: Keep startup minimal; avoid heavy IO on boot; lazy/open log file on first write where possible.
  - Test: T-CLI-PERF-001: Measure `duet-rpc --version` and `--help` complete within relaxed CI threshold (e.g., <200ms after warmup).

- [x] 11. Test suite scaffolding and golden setup
  - Code: Add `test` target; helper to spawn built exe via `typed-process`; per-test temp dirs via `temporary`. Store `test/golden/help_plain.txt` captured with `NO_COLOR=1 duet-rpc --help`.
  - Test: `cabal test` runs; E2E tests green for: T-CLI-VER-001/002, T-CLI-HLP-001/002, T-CLI-ERR-001, T-CLI-CLR-001/002; golden under VCS.

- [x] 12. Optional TTY color test (platform-gated)
  - Code: Add Unix-only test using `script -q -c 'duet-rpc --help' /dev/null` to emulate TTY; skip on Windows.
  - Test: T-CLI-TTY-001 (P1): TTY-attached help includes ANSI; piped/plain does not.

- [ ] 13. Unit tests for pure components
  - Code: Unit tests for VersionManager formatting; OutputFormatter precedence and ANSI absence when disabled; ErrorHandler “Use --help” hint.
  - Test: Unit suite green; 70–80%+ coverage on changed code; 100% on critical branches.

- [ ] 14. Final matrix and DoD
  - Code: N/A (verification step).
  - Test: All P0 tests pass; P1 covered where feasible; golden diffs reviewed/approved; no open Sev-1 defects.

---

## Definition of Done (Story 001)

- [ ] `duet-rpc --version` and `duet-rpc version` print the same semver.
- [ ] `duet-rpc --help` and `duet-rpc` show synopsis, subcommands, and footer.
- [ ] Unknown subcommand/flag → error + usage; no stack trace.
- [ ] Help output color rules: colorized on TTY; plain when piped; honors `NO_COLOR` and `--no-color`.
- [ ] Logging: default warn to stderr; `--log-level` applied; `DUET_RPC_LOG` routes to file; invalid path falls back to stderr with single warning.
- [ ] All outputs newline-terminated.
- [ ] Typical run responds within ~100ms (relaxed to ~200ms in CI).
- [ ] Tests (P0) are green; golden help approved.
