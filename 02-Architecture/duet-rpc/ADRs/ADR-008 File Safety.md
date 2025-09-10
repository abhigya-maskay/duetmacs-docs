---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-008]
---

# ADR-008: File Safety

**Status**: Accepted  
**Date**: 2025-01-05

## Context
- Tool modifies user files based on AI suggestions
- Must prevent data loss and allow recovery
- Security concerns about path traversal

## Options
- A: Dry-run mode with confirmation
- B: Temporary workspace
- C: Container isolation
- D: Permission checks only

## Trade-offs
- A: ✓ Simple, explicit ✗ No isolation
- B: ✓ Safe preview ✗ Complex workflow
- C: ✓ Full isolation ✗ Very complex
- D: ✓ Minimal ✗ Insufficient safety

## Decision Scope
- In scope: File modification safety, backups
- Out of scope: Network isolation

## Decision
Dry-run with confirmation (A) + path canonicalization + jail to project root

## Consequences
- Positive: Simple, effective safety
- Negative: Relies on user attention
- Follow-up: Implement .bak timestamped backups

## Open Questions
None

## References
- Epic: Safety & Controls

