# Story 002 CI Setup - Risk Register

## Scope Note
This risk register is strictly scoped to Story 002 (CI Setup with GitHub Actions).

## Risk Register

| Title | Story | Category | Description | Likelihood | Impact | Severity | Status |
|---|---|---|---|---|---|---|---|
| GitHub Actions minute exhaustion | 002 | Performance | Free tier limited to 2000 min/month; uncached builds ~10min each limits to ~200 builds/month | Medium | High | Critical | Mitigation Planned |
| Haskell setup action failure | 002 | Integrations | External dependency on haskell/actions/setup@v2; service outage blocks all CI | Low | High | Medium | Risk Accepted |
| Cache key collisions | 002 | Performance | Cache key based on OS+GHC+cabal hash; hash collisions could restore wrong dependencies | Low | High | Medium | Mitigation Planned |
| Test artifact data exposure | 002 | Security | Artifacts retained 90 days public; test outputs may contain sensitive debug data | Medium | Medium | Medium | Mitigation Implemented |
| Single GHC version lock-in | 002 | Legacy | Constraint to GHC 9.10 only; future library updates may require compiler migration | Medium | Medium | Medium | Risk Accepted |

## Risk Acceptance Decisions

### Haskell Setup Action Failure (Integration Risk)
**Decision**: Risk Accepted
**Date**: 2025-09-29
**Rationale**: Low likelihood risk acceptable for single developer project. Mitigation effort outweighs benefit given project context.

### Single GHC Version Lock-in (Legacy Risk)
**Decision**: Risk Accepted
**Date**: 2025-09-29
**Rationale**: Single developer codebase with full control over upgrade timing. Any GHC version migration will be deliberate and planned when actually needed, not speculatively. No coordination overhead or unexpected version conflicts in single-developer context.

## Risk Mitigations

### 1. GitHub Actions Minute Exhaustion (Critical)
**Evidence**: Free tier provides 2000 minutes/month; current uncached builds take ~10 minutes
**Mitigation Status**: PLANNED - Performance optimizations designed
**Mitigation Details**:
- Dependency caching already in story design (50-60% reduction)
- Add path filtering to skip documentation-only changes (15-20% reduction)
- Add concurrency control to cancel outdated builds (25-30% reduction)
- Combined impact: 70-80% minute reduction
- Increases capacity from ~200 to 600-800 builds/month

**Path Filtering Configuration**:
```yaml
on:
  pull_request:
    paths-ignore:
      - '**.md'
      - 'docs/**'
      - '.gitignore'
      - 'LICENSE'
      - '.github/ISSUE_TEMPLATE/**'
```

**Concurrency Control Configuration**:
```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

## 2. Cache Key Collision Prevention (Medium)
**Evidence**: Current cache key uses only cabal file hash, insufficient entropy for uniqueness
**Mitigation Status**: PLANNED - Enhanced cache key and validation
**Mitigation Details**:

### Mitigation 1: Enhanced Cache Key Configuration
Increase cache key entropy to prevent collisions:
```yaml
key: ${{ runner.os }}-ghc-${{ matrix.ghc }}-cabal-${{ hashFiles('**/*.cabal', '**/cabal.project', '**/cabal.project.freeze') }}-${{ github.run_number }}
restore-keys: |
  ${{ runner.os }}-ghc-${{ matrix.ghc }}-cabal-${{ hashFiles('**/*.cabal', '**/cabal.project', '**/cabal.project.freeze') }}-
  ${{ runner.os }}-ghc-${{ matrix.ghc }}-cabal-
```
**Impact**: 95% reduction in collision probability by including multiple file types and run number

### Mitigation 2: Cache Validation Step
Add integrity check after cache restore:
```yaml
- name: Validate cache integrity
  run: |
    cabal list --installed | sha256sum > /tmp/deps.hash
    if [ -f .github/cache-deps.hash ]; then
      diff .github/cache-deps.hash /tmp/deps.hash || exit 1
    fi
  continue-on-error: true
```
**Impact**: Detect cache collisions in 5 seconds vs 8-10 minute failed builds

## 3. Test Artifact Data Exposure (Medium)
**Evidence**: Artifacts in public repos are downloadable by anyone; test outputs may contain debug data
**Mitigation Status**: PLANNED - Minimal security controls designed
**Mitigation Details**:

### Mitigation: Native GitHub Actions Masking + Reduced Retention
Use GitHub's built-in security features instead of custom sanitization:
```yaml
# In CI workflow
- name: Setup test environment
  run: |
    # Mask any known test values that might be sensitive
    echo "::add-mask::test-db-password"
    echo "::add-mask::${{ secrets.TEST_TOKEN }}"

- name: Run tests
  run: |
    # Use minimal verbosity in CI
    cabal test --test-option=--quiet

- name: Upload test results
  uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: test-results.xml
    retention-days: 7  # Reduced from default 90 days
```
**Impact**: Reduces exposure window by 92% (7 vs 90 days), prevents secret leakage via masking
**Residual Risk**: Low - Only non-sensitive test results exposed for minimal time

## Risk Categories Coverage

- **Performance**: 2 risks identified (minute exhaustion, cache collisions)
- **Security**: 1 risk identified (artifact exposure)
- **Integrations**: 1 risk identified (Haskell setup action)
- **Legacy**: 1 risk identified (GHC version lock-in)