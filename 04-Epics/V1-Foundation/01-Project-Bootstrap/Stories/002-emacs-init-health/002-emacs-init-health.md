---
tags: [story, emacs, health-check, initialization, epic/v1-foundation]
aliases: [Story 002, Emacs Init Story, Health Check Story]
---

# Story 002: Emacs Init and Health Check

As an Emacs user, I want the Emacs package to load, expose a version command, and perform a basic health check so that I can verify the plugin is installed and can reach the CLI.

## Priority
- Must

## Dependencies
- Story 001 (CLI Version/Help Bootstrap)

## Non-Goals
- Editing/chat features or patch flows
- Full RPC protocol beyond a simple ping
- Packaging/release automation

## Business Rationale
- Confirms the Emacs surface can discover and invoke the CLI, reducing setup friction and providing a clear installation verification step.

## Acceptance Criteria (Given/When/Then)
- Given the package is installed, When I run `M-x duet-rpc-version`, Then it invokes the `duet-rpc` binary and displays its version, exiting cleanly.
- Given the package is installed, When I run `M-x duet-rpc-health`, Then it performs a basic connectivity check (e.g., calling `duet-rpc rpc --ping`) and shows "connected/ping OK" in the minibuffer.
- Given a missing or misconfigured CLI, When I run either command, Then I see a concise, actionable error with guidance to configure the CLI path.
- Given a health check exceeds the default timeout (3s), When I run `M-x duet-rpc-health`, Then it reports a timeout in the minibuffer.
- Given I run `M-x duet-refresh`, Then it performs a quick ping without full diagnostics.

## UI Behavior
- Error handling: Success and error messages are concise in the minibuffer; missing CLI messages include clear next steps to configure the CLI path.

## Assumptions / Open Questions
- Assumption: The CLI provides a `rpc --ping` or equivalent for health checks — Confidence: high — Impact if wrong: need a fallback check — Validation: align with CLI story for ping.
- Assumption: Emacs can locate the CLI via PATH or a configurable variable (e.g., `duet-rpc-cli-path`) — Confidence: high — Impact: user setup friction — Validation: document default + override.
- Open question: Preferred names for start/stop/health commands (e.g., `duet-rpc-start`, `duet-rpc-stop`, `duet-rpc-health`)? → UX specifies these names and adds `duet-rpc-version`, `duet-rpc-locate-cli`, and `duet-dispatch`.

## Defaults & Config
- defgroup: `duet-rpc`
- defcustom: `duet-rpc-cli-path` (file path; nil → use PATH)
- defcustom: `duet-rpc-health-timeout` (number; default 3 seconds)
- Helper command: `M-x duet-rpc-customize` opens the Customize group.

## Success Metrics
- `M-x duet-rpc-version` and `M-x duet-rpc-health` verified in manual smoke checks across typical platforms.
- Clear error surfaced and recovery guidance when CLI missing/unavailable.
- Control/log buffers behave as specified: control opens on start; logs do not auto-open; transient menu exposes actions; minibuffer copy concise, logs capture details.
