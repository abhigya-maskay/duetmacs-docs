---
tags: [story, ci, testing, automation, epic/v1-foundation, index]
aliases: [Story 008, CI Smoke Checks]
---

# Story 008: CI Smoke Checks

## Overview
Establishes continuous integration with automated smoke tests for the duet-rpc CLI and Emacs package.

## Story Documents
- **Main Specification**: [[008-ci-smoke-checks]]

## Purpose
Ensures code quality and prevents regressions by running automated tests on every commit and pull request.

## Key Deliverables
- GitHub Actions workflow setup
- Cross-platform testing (Linux, macOS, Windows)
- Smoke test suite
- Build verification
- Release artifact generation

## Test Coverage
- CLI commands (version, help, doctor)
- Basic functionality verification
- Cross-platform compatibility
- Performance benchmarks

---
*Navigation: [[00-Start-Here/README|Home]] > [[04-Epics]] > [[04-Epics/V1-Foundation/01-Project-Bootstrap/README|Project Bootstrap]] > [[04-Epics/V1-Foundation/01-Project-Bootstrap/Stories/README|Stories]] > Story 008*