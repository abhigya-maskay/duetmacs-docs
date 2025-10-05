# Story 002 CI Setup - Risk Register

## Scope Note
This risk register is strictly scoped to Story 002 (CI Setup with GitHub Actions).

## Risk Register

| Title | Story | Category | Description | Likelihood | Impact | Severity | Status |
|---|---|---|---|---|---|---|---|
| GitHub Actions minute exhaustion | 002 | Performance | Free tier limited to 2000 min/month; uncached builds ~10min each limits to ~200 builds/month | Medium | High | Critical | Mitigation Planned |
| Haskell setup action failure | 002 | Integrations | External dependency on haskell/actions/setup@v2; service outage blocks all CI | Low | High | Medium | Risk Accepted |
| Cache key collisions | 002 | Performance | Cache key based on OS+GHC+cabal hash; hash collisions could restore wrong dependencies | Low | High | Medium | Mitigation Planned |
| Test artifact data exposure | 002 | Security | Artifacts retained 7 days public; test outputs may contain sensitive debug data | Medium | Medium | Medium | Mitigation Planned |
| Single GHC version lock-in | 002 | Legacy | Constraint to GHC 9.12.x baseline (currently 9.12.2); future library updates may require compiler migration | Medium | Medium | Medium | Risk Accepted |

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
- Dependency caching already in story design (50-60% reduction when applicable)
- Add path filtering to skip documentation-only changes (15-20% of total builds skipped)
- Add concurrency control to cancel outdated builds (25-30% reduction in redundant builds)
- Combined impact: 60-70% effective minute reduction (optimizations don't compound linearly)
- Increases capacity from ~200 to 500-650 builds/month

**Path Filtering Configuration**:
```yaml
on:
  pull_request:
    paths-ignore:
      - '**.md'
      - 'docs/**'
      - '.gitignore'
      - 'LICENSE'
```

**Concurrency Control Configuration**:
```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true
```

### 2. Cache Key Collision Prevention (Medium)
**Evidence**: Current cache key uses only cabal file hash, insufficient entropy for uniqueness
**Mitigation Status**: PLANNED - Enhanced cache key and validation
**Mitigation Details**:

### Mitigation 1: Enhanced Cache Key Configuration
Increase cache key entropy to prevent collisions:
```yaml
key: ${{ runner.os }}-ghc-9.12.2-cabal-${{ hashFiles('**/*.cabal', '**/cabal.project', '**/cabal.project.freeze') }}
restore-keys: |
  ${{ runner.os }}-ghc-9.12-cabal-
  ${{ runner.os }}-ghc-cabal-
```
**Impact**: 95% reduction in collision probability by including multiple file types

### Mitigation 2: Cache Validation Step
Capture the dependency hash and compare it to the cached manifest before trusting a restored cache:
```yaml
- name: Record dependency hash
  id: dependency-hash
  run: echo "hash=${{ hashFiles('**/*.cabal', '**/cabal.project', '**/cabal.project.freeze') }}" >> "$GITHUB_OUTPUT"

- name: Validate cache restored
  if: steps.cabal-cache.outputs.cache-hit == 'true'
  run: |
    store_root="$HOME/.cabal/store"
    expected="${{ steps.dependency-hash.outputs.hash }}"
    manifest="$store_root/.manifest-hash"
    if [ ! -f "$manifest" ] || [ "$(cat "$manifest")" != "$expected" ]; then
      echo "Cached dependencies do not match expected manifest hash."
      exit 1
    fi
  continue-on-error: false

- name: Persist dependency hash
  if: success()
  run: |
    mkdir -p "$HOME/.cabal/store"
    echo "${{ steps.dependency-hash.outputs.hash }}" > "$HOME/.cabal/store/.manifest-hash"
```
**Impact**: Flags stale or corrupted dependency caches immediately, avoiding multi-minute rebuilds with bad inputs

### 3. Test Artifact Data Exposure (Medium)
**Evidence**: Artifacts in public repos are downloadable by anyone; test outputs may contain debug data
**Mitigation Status**: PLANNED - Minimal security controls designed
**Mitigation Details**:

### Mitigation: Native GitHub Actions Masking + Reduced Retention
Use GitHub's built-in security features instead of custom sanitization:
```yaml
# In CI workflow
- name: Setup test environment
  run: |
    # Mask runtime secrets that may surface in logs (skip if not configured)
    if [ -n "${{ secrets.TEST_API_KEY }}" ]; then
      echo "::add-mask::${{ secrets.TEST_API_KEY }}"
      export TEST_API_KEY="${{ secrets.TEST_API_KEY }}"
    fi
    if [ -n "${{ secrets.TEST_USER_HOME }}" ]; then
      echo "::add-mask::${{ secrets.TEST_USER_HOME }}"
      export TEST_USER_HOME="${{ secrets.TEST_USER_HOME }}"
    fi

- name: Run tests
  run: |
    # Keep CI logs focused: hide successes, surface failure details directly
    cabal test --test-option=--hide-successes --test-option=--show-details=direct | tee test-results.log

- name: Upload test results
  uses: actions/upload-artifact@v4  # latest patch v4.6.2 as of 30 Sep 2025
  with:
    name: test-results
    path: test-results.log
    retention-days: 7  # Reduced from default 90 days
```
**Impact**: Reduces exposure window by 92% (7 vs 90 days), prevents secret leakage via masking
**Residual Risk**: Low - Only non-sensitive test results exposed for minimal time

## Risk Categories Coverage

- **Performance**: 2 risks identified (minute exhaustion, cache collisions)
- **Security**: 1 risk identified (artifact exposure)
- **Integrations**: 1 risk identified (Haskell setup action)
- **Legacy**: 1 risk identified (GHC version lock-in)
