---
tags: [epic, planned, v1-foundation]
aliases: [03-Scoped Context, Scoped Context & Templates]
---

# Epic: 03-Scoped Context & Templates

Status: Planned
Phase: V1-Foundation

## Goal
Deterministic prompt assembly with selectable scopes and reusable templates/presets.

## Scope Boundary
### In Scope
- Context scope selection (region/buffer/files/project)
- Template/preset system for common prompts
- Deterministic prompt assembly
- .gitignore and exclude patterns
- Preview of assembled components

### Out of Scope
- AI-driven context discovery (V1.1)
- Semantic code analysis
- Cross-repository context
- Dynamic context learning
- Indexing or caching

## Dependencies
- Epic: Chat-to-Patch (uses assembled prompts)

## Acceptance Criteria
- User selects scope + preset in Emacs
- Preview shows included files/snippets
- CLI displays assembled components with --show-prompt
- Consistent output given same inputs
- Respects .gitignore and custom excludes

Links
- [[Epic Roadmap]]

---
*Navigation: [[00-Start-Here/README|Home]] > [[04-Epics]] > 03-Scoped Context*
