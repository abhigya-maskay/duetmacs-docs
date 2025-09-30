---
tags: [epics, roadmap, planning, implementation]
aliases: [Roadmap, Development Plan]
---

# Epics: AI IDE (Vertical Slices)

This roadmap captures the vertical-slice epic we are actively pursuing.

Note on naming
- codex: OpenAI's CLI tool.
- claude-code: Anthropic's CLI tool.
- duet-rpc: Our RPC/CLI binary used by Emacs and for one-shot commands, supporting both OpenAI and Anthropic providers.

## Epic: Project Bootstrap (CLI + Emacs Plugin)
- Goal: Establish a working skeleton for duet-rpc (our CLI) with shared infrastructure and Emacs integration hooks.
- Scope: Binary executable `duet-rpc`; language toolchain setup; subcommands limited to `version` plus global `--version`/`--help`; logging infrastructure with configurable destinations; color-aware output formatter honoring opt-out flags; config path constants documented for the loader skeleton; packaging notes for distribution.
- Dependencies: None.
- Acceptance:
  - `duet-rpc --version` prints the semantic version and exits 0.
  - `duet-rpc version` mirrors the same output via the command registry.
  - `duet-rpc --help` shows synopsis and available subcommands (`version`, `doctor`, `rpc`, `prompt` placeholders) routed through the shared formatter.
  - Invoking `duet-rpc` with no arguments prints help and exits 0.
  - Unknown subcommands surface optparse-applicative usage errors without stack traces.
  - Output honors `NO_COLOR`, `--no-color`, and TTY detection for color handling.
  - Logging defaults to warn level on stderr and supports `--log-level` overrides plus `DUET_RPC_LOG` redirection with graceful fallback when the destination is invalid.
- Phase: V1-Foundation.

---

## Phasing Summary
- V1-Foundation: Project Bootstrap (CLI + Emacs Plugin).
