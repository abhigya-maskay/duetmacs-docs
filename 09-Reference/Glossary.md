---
tags: [reference, glossary, terminology]
aliases: [Terms, Terminology, Definitions]
---

# Glossary

Scope: Story 001 only (CLI Version/Help Bootstrap). Terms below are limited to what’s needed for the minimal CLI, logging, and config skeleton.

## Project Terms

### duet-rpc
The CLI binary for DuetMacs. In Story 001, it supports version and help commands with basic logging and output formatting.

### DuetMacs
The broader environment pairing Emacs UI with `duet-rpc`. Mentioned for context only in Story 001.

## Configuration Terms

### TOML
Configuration file format. Story 001 defines paths and precedence only; no parsing yet.

### Precedence
Order of priority for configuration sources in Story 001: CLI flags > environment > project > user > defaults.

### XDG
Cross-Desktop Group spec for user config paths (e.g., `~/.config/duet-rpc/`).

## Logging Terms

### Katip
Haskell logging library used for structured logs. Story 001 uses minimal levels and destinations.

---
*Navigation: [[00-Start-Here/README|Home]] > [[09-Reference]] > Glossary*
