---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-007]
---

# ADR-007: Data Persistence

**Status**: Accepted  
**Date**: 2025-01-05

## Context
- Need to persist chat sessions; configuration handled separately
- Want simple, debuggable storage
- No complex querying requirements

## Options
- A: JSON files in project directory
- B: SQLite database
- C: Directory structure with markdown
- D: In-memory only

## Trade-offs
- A: ✓ Simple, portable, debuggable ✗ No queries
- B: ✓ Queryable, ACID ✗ Binary format, dependency
- C: ✓ Human-readable ✗ Complex parsing
- D: ✓ Simplest ✗ No persistence

## Decision Scope
- In scope: Session storage
- Out of scope: Config storage (TOML, defined elsewhere), caching, metrics storage

## Decision
Sessions: JSON files in project directory (A). Config: TOML.

## Consequences
- Positive: Simple, version-control friendly
- Negative: Manual indexing for search
- Follow-up: Define JSON schemas; define TOML config schema and precedence

## Open Questions
None
