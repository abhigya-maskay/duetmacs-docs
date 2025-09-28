---
tags: [workflow, practices, standards]
aliases: [Practices, Standards, Guidelines]
---

# Cross-Cutting Practices

## Overview

These practices apply throughout all phases of the [[Feature Development Workflow]], ensuring consistent quality and maintainability.

## Code Quality Standards

### Code Conventions
- **Follow existing patterns**: Match the style of surrounding code
- **Use project libraries**: Don't reinvent existing utilities
- **Naming consistency**: Use established naming conventions
- **Type safety**: Leverage type systems fully

### Documentation Standards
#### Documentation Levels
| Level      | Location       | Content                 |
| ---------- | -------------- | ----------------------- |
| **Inline** | Code comments  | Why, not what           |
| **Module** | File headers   | Purpose and usage       |
| **API**    | Interface docs | Contracts and examples  |
| **Design** | Separate docs  | Decisions and rationale |

#### Documentation Principles
1. **Document decisions close to code**
2. **Explain "why" over "what"**
3. **Include examples for complex APIs**
4. **Keep documentation versioned with code**

### Code Review Checklist

#### Before Submission
- [ ] Tests pass locally
- [ ] Linting passes
- [ ] Documentation updated
- [ ] Commit messages clear
- [ ] No commented-out code
- [ ] No debug statements

#### Review Focus Areas
1. **Correctness**: Logic and edge cases
2. **Clarity**: Readability and naming
3. **Consistency**: Pattern adherence
4. **Coverage**: Test completeness
5. **Security**: Input validation



## Security Best Practices

### Security Checklist

#### Input Handling
- [ ] Validate all external input
- [ ] Sanitize before output
- [ ] Use parameterized queries
- [ ] Limit request sizes
- [ ] Rate limit APIs

#### Authentication & Authorization
- [ ] Verify permissions
- [ ] Use secure sessions
- [ ] Implement timeout
- [ ] Log security events
- [ ] Fail securely

#### Data Protection
- [ ] Encrypt sensitive data
- [ ] Use secure connections
- [ ] Mask in logs
- [ ] Implement backups
- [ ] Control access

### Common Security Pitfalls

1. **Never trust user input**
2. **Don't log sensitive data**
3. **Avoid eval/exec with user data**
4. **Don't store secrets in code**
5. **Always use HTTPS**

## Testing Discipline

### Test Pyramid

```
        /\
       /E2E\      <- 10% (Critical paths)
      /------\
     /  Integ  \   <- 30% (Component boundaries)
    /------------\
   /     Unit     \ <- 60% (Business logic)
  /----------------\
```

### Testing Guidelines

#### What to Test
- Business logic
- Edge cases
- Error conditions
- Integration points

#### What Not to Test
- Framework code
- Trivial getters/setters
- Configuration
- Third-party libraries
- UI layout

### Test Quality Indicators

Good tests are:
- **Fast**: Run in milliseconds
- **Isolated**: No dependencies
- **Repeatable**: Same result every time
- **Self-validating**: Clear pass/fail
- **Timely**: Written with code

## Incremental Development

### Incremental Principles

1. **Small, complete changes**: Each commit should work
2. **Feature flags**: Hide incomplete features
3. **Backward compatibility**: Don't break existing code
4. **Continuous integration**: Merge frequently
5. **Early feedback**: Demo often

### Incremental Strategies

| Strategy | When to Use | Implementation |
|----------|------------|----------------|
| **Feature Toggle** | Long-running features | Configuration flag |
| **Branch by Abstraction** | Refactoring core code | Parallel implementation |
| **Expand-Contract** | API changes | Deprecation cycle |
| **Dark Launch** | Risky features | Silent rollout |

## Communication Patterns

### Documentation Artifacts

| Artifact | Audience | Update Frequency |
|----------|----------|------------------|
| **Commit Messages** | Developers | Every commit |
| **PR Description** | Reviewers | Per PR |
| **Release Notes** | Users | Per release |
| **Design Docs** | Team | Major changes |
| **Runbooks** | Operations | As needed |

### Commit Message Format

```
type(scope): subject

body

footer
```

Types: feat, fix, docs, style, refactor, test, chore

### Pull Request Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing complete

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review complete
- [ ] Documentation updated
- [ ] No new warnings
```

## Monitoring & Observability

### Logging Standards

#### Log Levels
| Level | Use Case | Example |
|-------|----------|---------|
| ERROR | Failures requiring attention | Database connection lost |
| WARN | Recoverable issues | Retry succeeded |
| INFO | State changes | User logged in |
| DEBUG | Diagnostic information | Variable values |

#### Logging Best Practices
1. **Structured logging**: Use consistent format
2. **Correlation IDs**: Track requests
3. **Appropriate levels**: Don't spam logs
4. **Useful context**: Include relevant data
5. **No sensitive data**: Mask passwords, keys

### Metrics to Track

- Request rate
- Error rate and types
- Resource utilization
- Business metrics
- User actions

## Related Documents

- [[Feature Development Workflow]] - Main workflow
- [[Decision Points]] - Decision framework
- [[Test Strategy]] - Testing details
- [[Security Guidelines]] - Security specifics


---
*Navigation: [[00-Start-Here/README|Home]] > [[01-Process-and-Workflow]] > Cross-Cutting Practices*
