---
tags: [architecture, adr, duet-rpc]
aliases: [ADR-012]
---

# ADR-012: API Introspection & Versioning

**Status**: Accepted  
**Date**: 2025-10-03

## Context
- Story 003 introduces `listMethods` and `describeMethods` for discoverability.
- We need clear expectations for their stability and for versioning of the RPC surface.

## Options
- Human-readable introspection vs. machine-validated schemas (JSON Schema/OpenAPI) in v1.
- Strict stability guarantees for introspection text vs. informational only.
- Method naming with or without namespace prefixes.

## Decision Scope
- In scope: Introspection outputs, ordering, stability posture, SemVer policy, deprecation process, naming.
- Out of scope: Transport/framing (ADR-010), dispatcher behavior (ADR-011).

## Decision
- Introspection: Provide `listMethods` (name + human description) and `describeMethods` (name + human param/return strings). These are advisory; not a machine contract.
- Ordering: Return methods in a stable, deterministic order (lexicographic by name).
- Versioning: Use strict SemVer for the daemon’s RPC API surface. Breaking RPC changes require a major version bump; additive non-breaking changes are minor; fixes are patch.
- Additions: Adding new methods or optional parameters is allowed in minor releases if existing behavior is unchanged.
- Deprecation: Mark deprecated methods in descriptions and/or a `deprecated: true` hint; remove only after ≥1 minor release with CHANGELOG notice.
- Namespacing: Use plain method names in v1. Reserve `duet.`-prefixed names for future internal/reserved use.
- Negotiation: No protocol negotiation beyond `initialize` returning `protocolVersion: "2.0"`.

## Consequences
- Clients can discover capabilities while relying on SemVer for breaking-change signaling.
- Avoids premature schema commitments while keeping room to formalize later.

## Open Questions
None

## References
- Story 003: `03-Epics/V1-Foundation/01-Project-Bootstrap/Stories/003-json-rpc-daemon/003-json-rpc-daemon.md`
- ADR-002: RPC Protocol

