# Story 002: CI Setup - Implementation Checklist

## Overview

This checklist breaks down the CI setup story into atomic, validatable implementation steps. Each step can be completed independently and verified immediately. The workflow follows FCIS principles: the CI pipeline itself is an imperative shell (IO-heavy: checkout, build, test, cache, upload), so we build it incrementally from minimal viable checks to full requirements.

## Implementation Strategy

**Build Order**: MVP → Quality → Artifacts → Caching → Optimizations

Each phase adds functionality without disrupting previous phases. Every step includes explicit validation criteria to ensure correctness before proceeding.

---

## Phase 1: MVP Foundation (Must Have - AC1, AC2)

### Step 1.1: Create Workflow Directory Structure
- [x] Create `.github/` directory in repository root
- [x] Create `.github/workflows/` subdirectory
- **Validation**: Directories exist at expected paths
- **Commands**:
  ```bash
  mkdir -p .github/workflows
  ls -la .github/workflows
  ```

### Step 1.2: Create Minimal CI Workflow (Build Only)
- [x] Create `.github/workflows/ci.yml`
- [x] Add workflow name: "CI"
- [x] Configure trigger: `pull_request` events (opened, synchronize, reopened)
- [x] Define single job: `ci-checks` running on `ubuntu-latest`
- [x] Add checkout step: `actions/checkout@v4`
- [x] Add Haskell setup: `haskell/actions/setup@v2` with GHC 9.10.3, Cabal latest
- [x] Add build step: `cabal build --enable-tests --enable-benchmarks`
- **Validation**:
  - YAML syntax is valid: `yamllint .github/workflows/ci.yml` (or GitHub's parser)
  - File structure matches expected format
  - No syntax errors when workflow is pushed
- **Test Method**: Push to branch, verify workflow appears in Actions tab

### Step 1.3: Add Test Execution Step
- [x] Add test step after build: `cabal test`
- [x] Ensure step fails workflow if tests fail (default behavior)
- **Validation**:
  - Tests execute after build
  - Test failures cause workflow to fail
  - Test output visible in GitHub Actions logs
- **Test Method**: Introduce failing test temporarily, verify workflow fails

### Step 1.4: Verify PR Status Reporting
- [ ] Push workflow to feature branch
- [ ] Open pull request against main branch
- [ ] Observe GitHub PR checks section shows CI status
- [ ] Verify branch protection can be configured to require CI check
- **Validation**:
  - PR page displays "CI" check with pass/fail status
  - Check details link to workflow run
  - Failed check prevents merge when protection enabled
- **Test Method**: Create actual PR, inspect checks UI

---

## Phase 2: Code Quality (Should Have - AC3, AC4)

### Step 2.1: Add HLint Installation and Execution
- [ ] Add HLint installation step (use `cabal install hlint` or `apt-get`)
- [ ] Add HLint execution step: `hlint src/ app/`
- [ ] Configure to fail on any HLint warnings
- **Validation**:
  - HLint runs on all Haskell source files
  - Violations appear in CI logs with file:line references
  - Violations cause workflow to fail
- **Test Method**: Introduce intentional lint violation, verify detection

### Step 2.2: Add Ormolu Installation and Format Check
- [ ] Add Ormolu installation step (use `cabal install ormolu`)
- [ ] Add Ormolu check step: `ormolu --mode check $(find src app -name '*.hs')`
- [ ] Configure to fail on formatting violations
- **Validation**:
  - Ormolu checks all Haskell files
  - Improperly formatted files listed in output
  - Formatting violations cause workflow to fail
- **Test Method**: Introduce formatting violation, verify detection

---

## Phase 3: Test Artifacts (Should Have - AC5)

### Step 3.1: Capture Test Output with Focused Verbosity
- [ ] Modify test step to pipe output: `cabal test --test-option=--hide-successes --test-option=--show-details=direct | tee test-results.log`
- [ ] Ensure pipeline preserves exit code: `set -euo pipefail` in step
- **Validation**:
  - `test-results.log` file created in workspace
  - Log contains failure details but not success spam
  - Test failures still cause step to fail
- **Test Method**: Run tests, verify log exists and has expected content

### Step 3.2: Add Secret Masking Setup
- [ ] Add step before tests: "Setup test environment"
- [ ] Mask `TEST_API_KEY` if set: `echo "::add-mask::${{ secrets.TEST_API_KEY }}"`
- [ ] Mask `TEST_USER_HOME` if set: `echo "::add-mask::${{ secrets.TEST_USER_HOME }}"`
- [ ] Handle unset secrets gracefully with conditional checks
- **Validation**:
  - Workflow runs successfully even when secrets not configured
  - When secrets are set, values don't appear in logs
- **Test Method**: Inspect logs for masked values (if secrets configured)

### Step 3.3: Upload Test Artifacts with 7-Day Retention
- [ ] Add step after tests: `actions/upload-artifact@v4`
- [ ] Configure artifact name: `test-results`
- [ ] Configure path: `test-results.log`
- [ ] Set retention: `retention-days: 7`
- [ ] Use `if: always()` to upload even on test failure
- **Validation**:
  - Artifact appears in workflow run page under "Artifacts"
  - Artifact is downloadable
  - Artifact contains expected test output
  - Retention is 7 days (visible in UI)
- **Test Method**: Download artifact, verify content and retention

---

## Phase 4: Dependency Caching (Should Have - AC6)

### Step 4.1: Add Basic Dependency Caching with Enhanced Keys
- [ ] Add cache step before build: `actions/cache@v4` with `id: cabal-cache`
- [ ] Configure path: `~/.cabal/store`
- [ ] Configure cache key: `${{ runner.os }}-ghc-9.10-cabal-${{ hashFiles('**/*.cabal', '**/cabal.project', '**/cabal.project.freeze') }}`
- [ ] Configure restore-keys (fallback hierarchy):
  ```yaml
  restore-keys: |
    ${{ runner.os }}-ghc-9.10-cabal-
    ${{ runner.os }}-ghc-cabal-
  ```
- [ ] Add save cache step at end of workflow
- **Validation**:
  - First run: cache miss, builds all deps, saves cache
  - Second run: cache hit, skips dep downloads, faster build
  - Cache key includes OS, GHC version, and all cabal file hashes
- **Test Method**: Run workflow twice, compare build times and cache logs

### Step 4.2: Add Cache Integrity Validation
- [ ] Add step before cache restore: "Record dependency hash"
  - Capture hash: `echo "hash=${{ hashFiles('**/*.cabal', '**/cabal.project', '**/cabal.project.freeze') }}" >> "$GITHUB_OUTPUT"`
  - Set output ID: `dependency-hash`
- [ ] Add step after cache restore: "Validate cache restored"
  - Run only if cache hit: `if: steps.cabal-cache.outputs.cache-hit == 'true'`
  - Check manifest file: `$HOME/.cabal/store/.manifest-hash`
  - Compare with expected hash
  - Fail if mismatch: `exit 1`
  - Set `continue-on-error: false`
- [ ] Add step after successful build: "Persist dependency hash"
  - Run only on success: `if: success()`
  - Write hash to manifest: `echo "${{ steps.dependency-hash.outputs.hash }}" > "$HOME/.cabal/store/.manifest-hash"`
- **Validation**:
  - Cache hit with matching hash: validation passes
  - Cache hit with mismatched hash: validation fails, rebuild triggered
  - First run: manifest created for future validation
- **Test Method**: Manually corrupt cache manifest, verify validation catches it

---

## Phase 5: Workflow Optimizations (Optional/Stretch)

### Step 5.1: Add Path Filtering for Documentation Changes
- [ ] Add `paths-ignore` to pull_request trigger:
  ```yaml
  paths-ignore:
    - '**.md'
    - 'docs/**'
    - '.gitignore'
    - 'LICENSE'
  ```
- **Validation**:
  - Commit with only .md changes: workflow skipped
  - Commit with code + .md changes: workflow runs
- **Test Method**: Create PR with doc-only changes, verify skip

### Step 5.2: Add Concurrency Control to Cancel Outdated Runs
- [ ] Add concurrency configuration at workflow level:
  ```yaml
  concurrency:
    group: ${{ github.workflow }}-${{ github.ref }}
    cancel-in-progress: true
  ```
- **Validation**:
  - Push commit to PR: new workflow starts
  - Immediately push second commit: first run cancels
- **Test Method**: Rapid successive commits, verify cancellation in Actions UI

### Step 5.3: Add CI Status Badge to README
- [ ] Wait for CI to pass on main branch
- [ ] Add badge to README.md:
  ```markdown
  ![CI](https://github.com/{owner}/{repo}/actions/workflows/ci.yml/badge.svg)
  ```
- **Validation**:
  - Badge displays in README
  - Badge shows current CI status (passing/failing)
- **Test Method**: View README on GitHub, verify badge appearance

---

## Validation Summary

Each phase must be validated before proceeding to the next:

| Phase | Validation Method | Success Criteria |
|-------|------------------|------------------|
| 1.1   | Directory listing | Directories exist |
| 1.2   | YAML parse, Actions UI | Workflow appears, syntax valid |
| 1.3   | Test execution logs | Tests run, failures block |
| 1.4   | PR checks UI | Status visible, merge blocked on fail |
| 2.1   | Lint violation test | HLint detects issues, fails check |
| 2.2   | Format violation test | Ormolu detects issues, fails check |
| 3.1   | Log file inspection | Focused output captured |
| 3.2   | Log inspection | Secrets masked (if configured) |
| 3.3   | Artifact download | Artifact exists, correct content |
| 4.1   | Build time comparison | Cache hit reduces build time |
| 4.2   | Corrupt manifest test | Validation catches corruption |
| 5.1   | Doc-only PR | Workflow skipped |
| 5.2   | Rapid commits | Older runs cancel |
| 5.3   | README view | Badge displays correctly |

---

## Definition of Done

### Must Have (Story cannot close without these)
- [ ] Steps 1.1 through 1.4 completed and validated
- [ ] CI workflow triggers on pull_request events
- [ ] Build compiles project with GHC 9.10.3
- [ ] Tests run and block merge on failure
- [ ] Status checks visible on PR

### Should Have (Enhancements per AC3-AC6)
- [ ] Steps 2.1-2.2 completed (HLint, Ormolu)
- [ ] Steps 3.1-3.3 completed (Test artifacts, masking)
- [ ] Steps 4.1-4.2 completed (Caching with validation)
- [ ] All should-have features validated via actual PR run

### Optional (Stretch, nice to have)
- [ ] Steps 5.1-5.3 completed (Optimizations, badge)
- [ ] Documentation link to successful PR run with all checks
