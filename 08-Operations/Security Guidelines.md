---
tags: [operations, security, planned]
aliases: [Security Guidelines, Security Best Practices]
---

# Security Guidelines

**Status**: Planned (Design Phase)
Scope: Story 001 only (CLI Version/Help Bootstrap)

## Overview
For Story 001, security focuses on logging hygiene and safe, minimal CLI behavior. No network access, credential handling, or file modifications are in scope.

## Scope (Story 001)

### Logging Hygiene
- Do not log sensitive data (secrets, file contents, tokens)
- Sanitize environment variables and paths in log output
- On `DUET_RPC_LOG` failure, warn once; avoid leaking full paths if sensitive

### CLI Behavior
- No network calls or external integrations
- Error messages are concise and sanitized; no stack traces

### Security Checklist (Story 001)
- [ ] Error messages sanitized (no stack traces)
- [ ] Logs free of sensitive data
- [ ] File logging fallback to stderr on failure

## Related Documentation
- [[08-Operations/Error Handling|Error Handling]] - Secure error messages
- [[01-Process-and-Workflow/Cross-Cutting Practices|Cross-Cutting Practices]] - General security hygiene

---
*Navigation: [[00-Start-Here/README|Home]] > [[08-Operations]] > Security Guidelines*
