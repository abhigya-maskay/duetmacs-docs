---
tags: [operations, performance, benchmarks, planned]
aliases: [Performance Targets, Performance Requirements]
---

# Performance Targets

**Status**: Planned (Design Phase)
Scope: Story 001 only (CLI Version/Help Bootstrap)

## Overview
For Story 001, performance targets cover only `--version`, `version`, and `--help`. Targets reflect fast startup and responsive help output on typical developer hardware.

## Targets (Story 001)

### CLI operations
| Operation | Target | Maximum | Notes |
|-----------|--------|---------|-------|
| `--version` | <50ms | 100ms | Cold start |
| `version` | <50ms | 100ms | Cold start |
| `--help` | <100ms | 200ms | Includes synopsis + subcommands |

### Startup
- CLI usable in <100ms on typical developer hardware

## Measurement Strategy
- Manual local timing during development
- CI smoke check invokes `--version`/`--help` and asserts completion

## Related Documentation
- [[02-Architecture/duet-rpc/duet-rpc Technical Architecture|Technical Architecture]] - Performance design
- [[07-Testing/Test Strategy|Test Strategy]] - Timing checks for bootstrap commands
- [[08-Operations/Observability|Observability]] - Logging only (no metrics in Story 001)

---
*Navigation: [[00-Start-Here/README|Home]] > [[08-Operations]] > Performance Targets*
