---
tags: [story, cli, rpc, ping, connectivity, epic/v1-foundation, index]
aliases: [Story 004, RPC Ping]
---

# Story 004: CLI RPC Ping

## Overview
Implements the `duet-rpc rpc --ping` command for testing RPC connectivity.

## Story Documents
- **Main Specification**: [[004-cli-rpc-ping]]
- **UX Specification**: [[ux]]

## Purpose
Provides a simple ping/pong mechanism to verify RPC communication between Emacs and the duet-rpc backend.

## Key Deliverables
- `duet-rpc rpc --ping` command
- Timeout handling
- RPC handshake verification (protocol specifics TBD)
- Response time measurement
- Both text and JSON output formats

---
*Navigation: [[00-Start-Here/README|Home]] > [[04-Epics]] > [[04-Epics/V1-Foundation/01-Project-Bootstrap/README|Project Bootstrap]] > [[04-Epics/V1-Foundation/01-Project-Bootstrap/Stories/README|Stories]] > Story 004*
