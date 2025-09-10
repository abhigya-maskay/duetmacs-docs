---
tags: [epic, planned, v1-foundation]
aliases: [02-Chat to Patch, Chat-to-Patch Epic]
---

# Epic: 02-Chat to Patch

Status: Planned
Phase: V1-Foundation

## Goal
Minimal end-to-end chat that produces a single-file patch with review/apply and backups.

## Scope Boundary
### In Scope
- Single-file patch generation only
- Basic chat session management
- Diff review and apply/rollback
- Automatic backups before changes
- Parse code blocks from AI responses

### Out of Scope  
- Multi-file patches (see V1.1)
- Advanced context assembly (see Epic 03)
- Provider switching (see Epic: Provider Switch)
- Persistence/history (separate epic)
- Search/discovery features

## Dependencies
- Epic: Project Bootstrap (CLI foundation)

## Acceptance Criteria
- From Emacs: start chat, request file edit, see diff, apply or rollback
- From CLI: pass file/prompt, receive patch, apply with backup
- Code blocks display with target file paths
- Status and token counters visible

Links
- [[Epic Roadmap]]

---
*Navigation: [[00-Start-Here/README|Home]] > [[04-Epics]] > 02-Chat to Patch*
