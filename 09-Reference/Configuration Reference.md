---
tags: [reference, configuration, toml, planned]
aliases: [Configuration Reference, Config Reference]
---

# Configuration Reference

**Status**: Planned (Design Phase)
Scope: Story 001 only (CLI Version/Help Bootstrap)

## Overview
For Story 001, configuration is a skeleton: define search paths and precedence only. No file I/O or parsing is performed yet.

## Configuration Files (Story 001)

### File Locations
1. **User** — `~/.config/duet-rpc/config.toml` (XDG)
2. **Project** — `.duet-rpc.toml` (project root)
3. **Custom** — Via `DUET_RPC_CONFIG` environment variable

### Precedence Order (Story 001)
Higher priority overrides lower:
1. Command-line flags (e.g., `--log-level`, `--no-color`)
2. Environment variables (e.g., `DUET_RPC_LOG`, `NO_COLOR`, `DUET_RPC_CONFIG`)
3. Project config file
4. User config file
5. Built-in defaults

## Configuration Format (TOML)

Story 001 does not parse or load configuration files. The following example illustrates future structure; only logging- and color-related keys are relevant to Story 001.

```toml
[general]
log_level = "warn"    # used when config loading is implemented
color_output = true    # used when config loading is implemented
```

### Environment Variables (Story 001)
- `DUET_RPC_LOG` — Log file path (falls back to stderr on error)
- `DUET_RPC_CONFIG` — Config file path (recognized path; not parsed in Story 001)
- `NO_COLOR` — Disable colors

### Validation Rules (Story 001)
- No file I/O or parsing; validation deferred to later stories

## Related Documentation
- [[05-Components/CLI-Components/04-config-loader|Config Loader Component]] - Implementation details
- [[02-Architecture/duet-rpc/duet-rpc Technical Architecture|Technical Architecture]] - Config design decisions

---
*Navigation: [[00-Start-Here/README|Home]] > [[09-Reference]] > Configuration Reference*
