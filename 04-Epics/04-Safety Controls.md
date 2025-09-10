---
tags: [epic, planned, v1-foundation]
aliases: [04-Safety Controls, Safety & Controls]
---

# Epic: 04-Safety & Controls

Status: Planned
Phase: V1-Foundation

## Goal
Guardrails consistently applied across UI and CLI for writes and long operations.

## Scope Boundary
### In Scope
- Dry-run mode toggle
- Write allowlist prompts
- File/size caps with warnings
- Automatic backups before changes
- Restore/rollback functionality
- Truncation with clear notices

### Out of Scope
- Semantic safety checks
- AI-based risk assessment
- Network security controls
- Sandboxing or containerization
- Version control integration

## Dependencies
- Epic: Chat-to-Patch (applies controls to patches)

## Acceptance Criteria
- All patch applications require explicit approval
- Over-cap inputs are summarized with clear messaging
- Backup/restore verified in both UI and CLI
- Dry-run mode prevents all writes
- Size/file count limits enforced consistently

Links
- [[Epic Roadmap]]

---
*Navigation: [[00-Start-Here/README|Home]] > [[04-Epics]] > 04-Safety Controls*
