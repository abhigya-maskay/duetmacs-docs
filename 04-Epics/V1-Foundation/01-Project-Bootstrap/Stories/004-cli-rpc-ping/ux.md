---
tags: [ux, story/004, cli, rpc, ping]
aliases: [Story 004 UX, RPC Ping UX, CLI Ping UX]
---

# Story 004: CLI RPC Ping

UX level: Light, specify output, flags, errors, examples.

## Summary
- Default success prints `pong` with trailing newline to stdout.
- JSON mode prints minimal object with `status`, `version`, `timestamp` to stdout.
- Timeouts and errors produce a concise message to stderr. In `--json` mode, errors emit JSON.

## Output Formats
- Text (default):
  - Success: `pong\n` (stdout)
  - No additional text, version, or timestamp in text mode
- JSON (`--json`):
  - Success (stdout):
    ```json
    {"status":"ok","version":"<semver>","timestamp":"<ISO-8601-UTC>"}
    ```

## Flags
- `--json`: switches output to JSON as defined above
- `--timeout <ms>`: maximum time in milliseconds to complete handshake
- Unknown/invalid flags: print helpful usage to stderr

## Errors
- Timeout:
  - stderr: `timeout exceeded after <ms> ms`
  - With `--json`: stdout (or stderr) emits JSON: `{"status":"error","message":"timeout exceeded after <ms> ms"}`; prefer stdout for machine parsing
- Generic error (handshake failure, transport error, etc.):
  - stderr: concise cause summary (no stack by default)
  - With `--json`: JSON object with fields:
    - `status`: `"error"`
    - `message`: concise human-readable message
    - Optional: `code` (string identifier) when available

## Streams
- Success output goes to stdout only
- Error details go to stderr in text mode; JSON error responses may go to stdout to ease parsing in scripts

## Examples
- Default success:
  - Command: `duet-rpc rpc --ping`
  - stdout: `pong\n`
- JSON success:
  - Command: `duet-rpc rpc --ping --json`
  - stdout: `{"status":"ok","version":"0.1.0","timestamp":"2025-01-15T12:34:56Z"}`
- Timeout (text):
  - Command: `duet-rpc rpc --ping --timeout 500`
  - stderr: `timeout exceeded after 500 ms`
- Timeout (json):
  - Command: `duet-rpc rpc --ping --timeout 500 --json`
  - stdout: `{"status":"error","message":"timeout exceeded after 500 ms"}`
- Invalid flag:
  - Command: `duet-rpc rpc --ping --unknown`
  - stderr: usage with brief help for `--ping`

## Edge Cases
- If both stdout and stderr are produced in error mode, ensure JSON consumers can rely on a single JSON object on stdout; keep human-readable diagnostics on stderr
- Honor `--timeout` across all transport steps (startup, connect, handshake)
- Locale-independent outputs: fixed strings, ISO-8601 UTC timestamps
- Do not print ANSI color or formatting in JSON mode
