---
tags: [architecture, adr, index, duet-rpc]
aliases: [ADR Index, ADR Log, Decision Log]
---

# duet-rpc ADR Index

This index lists all Architecture Decision Records (ADRs) for duet-rpc and their status.

## Decision Table

| ADR | Area | Decision | Status |
|-----|------|----------|--------|
| [[ADR-001 Architecture Style|ADR-001]] | Architecture Style | Hybrid daemon + CLI | Accepted |
| [[ADR-002 RPC Protocol|ADR-002]] | RPC Protocol | JSON-RPC 2.0 over stdio | Accepted |
| [[ADR-003 Haskell Build System|ADR-003]] | Build System | Cabal with GHC 9.10 | Accepted |
| [[ADR-004 Core Libraries|ADR-004]] | Core Libraries | See detailed list | Accepted |
| [[ADR-005 Error Handling|ADR-005]] | Error Handling | Typed ADTs everywhere | Accepted |
| [[ADR-006 State Management|ADR-006]] | State Management | STM with TVar/TMVar | Accepted |
| [[ADR-007 Data Persistence|ADR-007]] | Persistence | Sessions: JSON in project dir; Config: TOML | Accepted |
| [[ADR-008 File Safety|ADR-008]] | File Safety | Dry-run + canonicalization | Accepted |
| [[ADR-009 Testing Strategy|ADR-009]] | Testing | Property + unit with tasty | Accepted |

---
*Navigation: [[00-Start-Here/README|Home]] > [[01-Architecture]] > [[duet-rpc/duet-rpc Technical Architecture|duet-rpc Architecture]] > ADRs*
