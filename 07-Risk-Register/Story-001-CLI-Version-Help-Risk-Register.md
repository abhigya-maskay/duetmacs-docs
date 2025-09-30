---
tags: [risk-management, story-001, cli]
aliases: [Story 001 Risk Register, Risk Register (Story 001), Story 001 Risks]
---

# Story 001 Risk Register (CLI Version/Help)

## Scope Note
This risk register is strictly scoped to Story 001 (CLI Version/Help Bootstrap).

## Risk Register

| Title | Story | Category | Description | Likelihood | Impact | Severity | Status |
|---|---|---|---|---|---|---|---|
| Cross-platform TTY detection | 001 | Legacy | TTY detection behaves differently across Windows/Unix; NO_COLOR handling may be inconsistent | Medium | Medium | Medium | In Progress |
| Log file permission denied | 001 | Security | DUET_RPC_LOG path may point to restricted directories; fallback to stderr not guaranteed to preserve all log data | Medium | Low | Low | Resolved |
| Version string mismatch | 001 | Legacy | Version sourced from package manifest; build-time vs runtime version may diverge in dev environments | Low | Low | Low | Resolved |
| Colorization terminal escape | 001 | Security | ANSI escape sequences in help output; potential for terminal injection if not properly sanitized | Low | High | Medium | Mitigation Planned |
| Unicode fallback symbols | 001 | Legacy | Unicode detection for symbols (OK/WARN/FAIL); may display incorrectly in some terminals | Low | Low | Low | Resolved |

## Critical Risks Summary

### 1. Cross-Platform TTY Detection (Medium)
**Evidence**: Multiple platform support implied but Windows-specific behavior not detailed
**Mitigation Status**: IN PROGRESS - [[04-Components/CLI-Components/01-output-formatter|OutputFormatter]] component fully designed
**Mitigation Details**:
- ansi-terminal library provides cross-platform support
- Windows VT enablement handled by library
- Fallback to ColorDisabled when VT unsupported
- Separate TTY detection for stdout and stderr
- Complete technical design in [[04-Components/CLI-Components/01-output-formatter|OutputFormatter]] component

### 2. Log File Permission Denied (Low)
**Evidence**: DUET_RPC_LOG may point to restricted directories
**Mitigation Status**: RESOLVED - Already in acceptance criteria
**Mitigation Details**:
- Acceptance criteria explicitly handles this case
- Falls back to stderr with single warning message
- No stack trace shown to user
- Command continues with normal operation

### 3. Version String Mismatch (Low)
**Evidence**: Version sourced from package manifest
**Mitigation Status**: RESOLVED - Single source of truth established
**Mitigation Details**:
- Cabal package version via generated Paths module
- [[04-Components/CLI-Components/02-version-manager|VersionManager]] component ensures consistency
- No manual version strings to maintain
- Build-time version automatically available at runtime


### 5. Colorization Terminal Escape (Medium)
**Evidence**: ANSI codes in help output without sanitization mention
**Mitigation Status**: PLANNED - Added to OutputFormatter component
**Mitigation Details**:
- Strip control characters except newline/tab
- Remove existing ANSI sequences from input
- Validate text before applying our own ANSI codes
- Added sanitization responsibility to [[04-Components/CLI-Components/01-output-formatter|OutputFormatter]]

### 6. Unicode Fallback Symbols (Low)
**Evidence**: Unicode detection for terminal symbols
**Mitigation Status**: RESOLVED - Complete design in [[04-Components/CLI-Components/01-output-formatter|OutputFormatter]]
**Mitigation Details**:
- DUET_RPC_ASCII environment variable for forcing ASCII
- UTF-8 detection from LANG/LC_* environment variables
- Explicit fallback symbols defined: OK→[OK], WARN→[WARN], etc.
- SymbolSet type with Unicode and ASCII variants implemented

## Risk Categories Coverage

- **Security**: 2 risks identified (log permissions, terminal escape)
- **Legacy**: 3 risks identified (cross-platform, version, unicode)

## Status Definitions
- **Assessed**: Risk has been identified and scored but no mitigation planned yet
- **Mitigation Planned**: Approach to address risk has been defined
- **In Progress**: Mitigation work is underway
- **Resolved**: Risk has been eliminated
- **Accepted**: Risk acknowledged but will not be mitigated
