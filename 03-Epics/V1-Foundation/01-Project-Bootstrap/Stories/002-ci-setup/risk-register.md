# Story 002 CI Setup - Risk Register

## Risk Register

| Title | Category | Description | Likelihood | Impact | Severity | Status |
|---|---|---|---|---|---|---|
| GitHub Actions minute exhaustion | Performance | Free tier limited to 2000 min/month; uncached builds ~10min each limits to ~200 builds/month | Medium | High | Critical | Assessed |
| Haskell setup action failure | Integrations | External dependency on haskell/actions/setup@v2; service outage blocks all CI | Low | High | Medium | Assessed |
| Cache key collisions | Performance | Cache key based on OS+GHC+cabal hash; hash collisions could restore wrong dependencies | Low | High | Medium | Assessed |
| Test artifact data exposure | Security | Artifacts retained 90 days public; test outputs may contain sensitive debug data | Medium | Medium | Medium | Assessed |
| Single GHC version lock-in | Legacy | Constraint to GHC 9.10 only; future library updates may require compiler migration | Medium | Medium | Medium | Assessed |
| PR-based cache poisoning | Security | Malicious PR could inject compromised dependencies into shared cache | Low | High | Medium | Assessed |

## Phase 2 Gate Checklist

- ✅ **Scope**: All categories assessed (Integrations, Performance, Security, Legacy)
- ✅ **Register**: 6 risks documented with required fields
- ✅ **Scoring**: Severity computed via matrix (1 Critical, 5 Medium)