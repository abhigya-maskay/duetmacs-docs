---
tags: [story, cli, bootstrap, epic/v1-foundation, index]
aliases: [Story 001, CLI Bootstrap]
---

# Story 001: CLI Version/Help Bootstrap

## Overview
Establishes the foundational CLI with version and help commands, plus core infrastructure components.

## Story Documents
- **Main Specification**: [[001-cli-version-help-bootstrap]]
- **UX Specification**: [[ux]]
- **Test Strategy**: [[test-strategy]]
- **Implementation Checklist**: [[implementation-checklist]]
- **Risk Register**: [[10-Risk-Register/Story-001-CLI-Version-Help-Risk-Register|Story 001 Risks]]

## Components
This story introduces the foundational CLI components (canonical specs in `05-Components/CLI-Components`):

- [[Output Formatter]] - Terminal output handling
- [[Version Manager]] - Version information management
- [[Logger]] - Structured logging infrastructure
- [[Config Loader]] - Configuration management skeleton
- [[Help Formatter]] - Help text generation
- [[Error Handler]] - Error formatting
- [[CLI Parser]] - Command-line argument parsing
- [[05-Components/CLI-Components/component-map|Component Map]] - Overview of all components

## Key Deliverables
- `duet-rpc --version` command
- `duet-rpc --help` command
- Logging infrastructure
- Output formatting with color support
- Error handling framework

## Acceptance Criteria
- Binary responds to version/help commands
- Color output respects TTY and NO_COLOR
- Structured logging to stderr/file
- ~100ms response time

---
*Navigation: [[00-Start-Here/README|Home]] > [[04-Epics]] > [[04-Epics/V1-Foundation/01-Project-Bootstrap/README|Project Bootstrap]] > [[04-Epics/V1-Foundation/01-Project-Bootstrap/Stories/README|Stories]] > Story 001*
