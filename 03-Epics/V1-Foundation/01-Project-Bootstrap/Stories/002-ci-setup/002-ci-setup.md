---
tags: [story, ci, github-actions, automation, testing]
aliases: [Story 002, CI Setup, GitHub Actions Setup]
---

# Story 002: CI Setup with GitHub Actions

## Story Overview

**Epic**: Project Bootstrap (CLI + Emacs Plugin)
**Story Number**: 002
**Story Name**: CI Setup with GitHub Actions
**Status**: Planning
**Priority**: Must Have
**Estimated Complexity**: Simple

## Business Context

### Objective
Establish automated build validation on every pull request to ensure code quality and prevent broken builds from entering the main branch.

### Success Metrics
- All PRs must have passing build status before merge capability is enabled
- Build feedback provided to developers via GitHub PR status checks
- No broken builds merged to main branch

### Target Users
- **Primary**: Developers submitting pull requests
- **Secondary**: Maintainers reviewing pull requests

### Business Value
- Prevents broken code from entering main branch
- Maintains code quality standards automatically
- Reduces manual review burden for build/test verification
- Provides rapid feedback to developers

## Scope Definition

### In Scope (Story 002)

#### Must Have Requirements
1. **Automated PR Builds**
   - Trigger on PR open/synchronize events
   - Build project with Cabal on Linux
   - Use GHC 9.10.3 (current stable release as of 30 Sep 2025)
   - Report build status to PR

2. **Automated Test Execution**
   - Run all Tasty unit tests
   - Block PR merge on test failures
   - Clear pass/fail status on PR

#### Should Have Requirements
3. **HLint Code Quality Checks**
   - Run HLint on all Haskell source files
   - Report linting issues
   - Fail PR check on violations

4. **Ormolu Formatting Validation**
   - Check code formatting with Ormolu
   - Detect formatting violations
   - Fail PR check on improper formatting

5. **Test Result Artifacts**
   - Generate test result reports
   - Capture `cabal test` output to `test-results.log`
   - Upload as GitHub Actions artifacts
   - Retain for 7 days (reduced from default 90 days for security)
   - Use focused test verbosity (--hide-successes with direct failure output)
   - Implement secret masking for any test values

6. **Dependency Caching**
   - Cache ~/.cabal/store directory
   - Enhanced key strategy: OS + GHC version + hashes of cabal, cabal.project, and cabal.project.freeze files
   - Validate cache integrity after restore with dependency hash check
   - Cache fallback keys use `${{ runner.os }}-ghc-9.10-cabal-` then `${{ runner.os }}-ghc-cabal-`
   - Expected 50-60% build time reduction

### Out of Scope (Explicit Non-Goals)
- Deployment or release automation
- Code coverage reporting
- Performance benchmarking
- Multi-OS testing (Windows, macOS)
- Multiple GHC version testing
- Security/vulnerability scanning
- Commit signing enforcement
- PR auto-merge capabilities
- Documentation generation
- Binary artifact publishing

## Technical Specifications

### Environment Requirements
- **Platform**: Linux (ubuntu-latest)
- **Language**: Haskell
- **Compiler**: GHC 9.10.3 (track latest 9.10.x patch release)
- **Build Tool**: Cabal
- **Test Framework**: Tasty (existing)
- **Code Quality**: HLint, Ormolu
- **CI Platform**: GitHub Actions (free tier)

### Repository Configuration
- **Hosting**: GitHub (public repository)
- **Branch Strategy**: Main branch with feature branches
- **Merge Strategy**: All changes via pull requests
- **Protection Rules**: Require CI checks to pass

### Implementation Approach
1. Single workflow file: `.github/workflows/ci.yml`
2. Triggered on pull_request events
3. Single job with multiple steps (build, test, lint, format)
4. Status checks reported back to PR
5. Artifacts uploaded for test results only

## Acceptance Criteria

### Must Have Acceptance Criteria

**AC1: Automated PR Builds**
- GIVEN a pull request is opened or synchronized
- WHEN the CI workflow triggers automatically
- THEN the project builds with Cabal using GHC 9.10.3 on Linux
- AND the build status (pass/fail) appears on the PR page
- AND build failures prevent PR merging

**AC2: Automated Test Execution**
- GIVEN the PR build completes successfully
- WHEN the test step executes
- THEN all Tasty unit tests run
- AND test results determine the PR check status
- AND any test failure blocks PR merge capability

### Should Have Acceptance Criteria

**AC3: HLint Code Quality**
- GIVEN Haskell source files in the PR
- WHEN the HLint step runs
- THEN code quality issues are identified
- AND violations cause the PR check to fail
- AND specific violations are visible in CI logs

**AC4: Ormolu Formatting Check**
- GIVEN Haskell source files in the PR
- WHEN the Ormolu check runs
- THEN formatting inconsistencies are detected
- AND improperly formatted files cause check failure
- AND affected files are listed in CI output

**AC5: Test Result Artifacts**
- GIVEN test execution completes (pass or fail)
- WHEN test results are generated
- THEN a test report is uploaded as an artifact
- AND the artifact is downloadable for 7 days
- AND the artifact includes test output with minimal verbosity
- AND any sensitive test values are masked

**AC6: Dependency Caching**
- GIVEN a CI workflow starts
- WHEN a matching cache exists (OS+GHC+cabal hash)
- THEN the ~/.cabal/store directory is restored
- AND dependency downloads are skipped
- AND build time is reduced significantly

## Dependencies and Constraints

### Dependencies
- GitHub Actions must be enabled on the repository
- Project must build successfully with GHC 9.10.3 locally (and future 9.10.x patches as they ship)
- Existing Tasty tests must be passing
- Cabal file must correctly specify all dependencies

### Constraints
- Limited to GitHub Actions free tier (2000 minutes/month)
- Linux-only testing environment
- Single GHC version (9.10.x baseline, currently 9.10.3)
- No external service dependencies
- Public repository visibility

## MVP Definition

**Minimal Viable Product for Story 002:**
- **User**: Developer creating a PR
- **Trigger**: PR opened or synchronized
- **Flow**: Build → Test → Status Report
- **Outcome**: PR shows pass/fail status, merge blocked on failure
- **Deferred**: Code quality checks can be added after MVP works

## Implementation Notes

### File Structure
```
.github/
└── workflows/
    └── ci.yml         # Main CI workflow definition
```

### Workflow Structure
1. **Trigger**: On pull_request (opened, synchronize, reopened)
   - Path filtering to skip documentation-only changes (**.md, docs/**, .gitignore, LICENSE)
   - Conserves GitHub Actions minutes (15-20% reduction)
   - Concurrency control to cancel outdated builds (25-30% minute reduction)
2. **Job**: ci-checks
3. **Steps**:
   - Checkout code
   - Setup Haskell (GHC 9.10.3, Cabal)
   - Restore cache with enhanced key (OS+GHC+cabal files hashes; restore keys fall back to `${{ runner.os }}-ghc-9.10-cabal-` then `${{ runner.os }}-ghc-cabal-`)
   - Validate cache integrity with dependency hash check
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
   - Build project
   - Setup test environment with secret masking (gracefully handles unset secrets)
     ```yaml
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
     ```
   - Run tests hiding successes while surfacing failure details
     ```yaml
     - name: Run tests
       run: |
         set -euo pipefail
         # Keep CI logs focused: hide successes, surface failure details directly
         cabal test --test-option=--hide-successes --test-option=--show-details=direct | tee test-results.log
     ```
   - Run HLint (should have)
   - Run Ormolu (should have)
   - Upload test artifacts with 7-day retention (reduced from 90 days)
   - Save cache with enhanced key strategy

### Configuration Decisions
- Use `haskell/actions/setup@v2` for Haskell setup
- Use `actions/cache@v4` (latest patch v4.3.0, published 24 Sep 2025) for dependency caching with enhanced keys (includes multiple cabal file types)
- Use `actions/upload-artifact@v4` (latest patch v4.6.2, as of 30 Sep 2025) for test results with 7-day retention
- Run all checks in single job to minimize overhead
- Fail fast on build, test, lint, and format errors so PR merge is blocked on any violation
- Enable path filtering and concurrency control to optimize GitHub Actions minute usage
- Implement cache validation to detect and handle cache corruption

## Definition of Done

### Must Have
Story 002 is complete when all mandatory items below are satisfied:
- [ ] CI workflow file created and committed
- [ ] Workflow triggers on pull_request open, synchronize, and reopen events
- [ ] Build step compiles the project with GHC 9.10.3 on ubuntu-latest runners
- [ ] Test step runs all existing Tasty tests (with failures blocking PR merge)
- [ ] Status checks report back to the PR and enforce branch protection requirements

### Should Have Enhancements (traceable to AC3–AC6)
The story may be closed with should-have scope once each acceptance criterion is satisfied:
- [ ] HLint check enforces AC3 (fails CI on reported issues, exposes findings in logs)
- [ ] Ormolu formatting validation enforces AC4 (fails CI on formatting drift, lists impacted files)
- [ ] Test artifact handling meets AC5 (uploads 7-day artifact, keeps logs quiet, masks sensitive values)
- [ ] Dependency caching fulfils AC6 (enhanced key, cache restore verification, measurable build-time reduction)

### Optional Workflow Optimizations (stretch)
Track separately so they do not block Definition of Done:
- [ ] Path filtering skips documentation-only changes to conserve CI minutes
- [ ] Concurrency control cancels in-progress runs when new commits arrive
- [ ] README carries a CI status badge once checks are green on main
- [ ] Should-have behaviours proven via at least one PR run (documentation link to run)

---
*Navigation: [[03-Epics/V1-Foundation/01-Project-Bootstrap/Stories|Stories]] > Story 002*
