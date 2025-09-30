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
   - Use GHC 9.10
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
   - Upload as GitHub Actions artifacts
   - Retain for 7 days (reduced from default 90 days for security)
   - Use minimal test verbosity (--quiet flag)
   - Implement secret masking for any test values

6. **Dependency Caching**
   - Cache ~/.cabal/store directory
   - Enhanced key strategy: OS + GHC version + hashes of cabal, cabal.project, and cabal.project.freeze files
   - Validate cache integrity after restore with dependency hash check
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
- **Compiler**: GHC 9.10
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
- THEN the project builds with Cabal using GHC 9.10 on Linux
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
- Project must build successfully with GHC 9.10 locally
- Existing Tasty tests must be passing
- Cabal file must correctly specify all dependencies

### Constraints
- Limited to GitHub Actions free tier (2000 minutes/month)
- Linux-only testing environment
- Single GHC version (9.10)
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
   - Setup Haskell (GHC 9.10, Cabal)
   - Restore cache with enhanced key (OS+GHC+cabal files hashes)
   - Validate cache integrity with dependency hash check
   - Build project
   - Setup test environment with secret masking
   - Run tests with --quiet flag for minimal verbosity
   - Run HLint (should have)
   - Run Ormolu (should have)
   - Upload test artifacts with 7-day retention (reduced from 90 days)
   - Save cache with enhanced key strategy

### Configuration Decisions
- Use `haskell/actions/setup@v2` for Haskell setup
- Use `actions/cache@v4` for dependency caching with enhanced keys (includes multiple cabal file types)
- Use `actions/upload-artifact@v4` for test results with 7-day retention
- Run all checks in single job to minimize overhead
- Fail fast on build errors, continue on test/lint errors for visibility
- Enable path filtering and concurrency control to optimize GitHub Actions minute usage
- Implement cache validation to detect and handle cache corruption

## Definition of Done

Story 002 is complete when:
- [ ] CI workflow file created and committed
- [ ] Workflow triggers on all PR events with path filtering for documentation-only changes
- [ ] Concurrency control implemented to cancel in-progress builds when new commits are pushed
- [ ] Build step compiles project with GHC 9.10
- [ ] Test step runs all Tasty tests with --quiet flag for minimal verbosity
- [ ] Secret masking configured for test environment values
- [ ] Build/test failures block PR merge
- [ ] HLint checks implemented (should have)
- [ ] Ormolu checks implemented (should have)
- [ ] Test artifacts uploaded with 7-day retention (should have)
- [ ] Dependency caching working with enhanced key strategy (cabal + cabal.project + cabal.project.freeze)
- [ ] Cache integrity validation implemented after restore
- [ ] README updated with CI badge
- [ ] All acceptance criteria verified on actual PR

---
*Navigation: [[03-Epics/V1-Foundation/01-Project-Bootstrap/Stories|Stories]] > Story 002*