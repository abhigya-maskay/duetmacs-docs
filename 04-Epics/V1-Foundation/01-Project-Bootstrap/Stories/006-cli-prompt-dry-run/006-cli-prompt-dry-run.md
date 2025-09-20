---
tags: [story, cli, prompt, dry-run, epic/v1-foundation]
aliases: [Story 006, Prompt Dry-Run Story, CLI Dry-Run]
---

# Story 006: CLI Prompt Dry-Run

As a CLI user, I want `duet-rpc prompt --file <path> --dry-run` to accept inputs and return a deterministic no-op response so that I can validate end-to-end wiring without making edits.

## Priority
- Must

## Dependencies
- Story 001 (CLI Version/Help Bootstrap)

## Non-Goals
- Real model/provider calls
- Patch generation or file writes
- Context assembly or safety caps beyond basic input validation

## Business Rationale
- Enables smoke tests and scripting of the full path from input parsing to output formatting without risking file modifications.

## Acceptance Criteria (Given/When/Then)
- Given a readable file path, When I run `duet-rpc prompt --file README.md --dry-run`, Then it prints a response indicating no-op with echoed inputs (file path, size, and content hash).
- Given `--json`, When I run the command, Then the output is a stable JSON structure containing fields like `mode: "dry-run"`, `file`, `size_bytes`, `sha256`, and `received_at`.
  - JSON timestamps are ISO-8601 UTC with trailing `Z`; key order is not significant; field names use `snake_case`.
- Given a usage error (e.g., missing `--file`), When I run the command, Then it prints a clear error with a one-line usage hint (e.g., "Run `duet-rpc prompt --help`.").
- Given a file I/O error (e.g., not found, permission), When I run the command, Then it prints a clear cause without a usage hint.
- Given extra flags like `--provider` or `--model`, When I run with `--dry-run`, Then they are accepted but ignored, with a notice that this is a no-op.
  - In text mode, warnings are written to stderr; in JSON mode, warnings are included in a `warnings` array.
  - Hashing uses SHA-256 over raw file bytes and outputs lowercase hex; the file path is echoed exactly as provided (no resolution).
  - If the file size exceeds 50 MB, the command still succeeds but emits a warning (stderr in text; `warnings[]` in JSON).

## Assumptions / Open Questions
- Assumption: File hashing uses SHA-256 — Confidence: high — Impact: deterministic validation — Validation: document and test in CI.
- Decision: `--stdin` (or `--file -`) is not supported for dry-run; attempts result in a usage error. Parity can be revisited later.

## Success Metrics
- Dry-run behaves deterministically across platforms and is used in CI smoke tests.
