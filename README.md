# Contribution [#1096]: Add test coverage for test-setup.ts utilities

**Contribution Number:** 2  
**Student:** Yifei Feng  
**Issue:** [GitHub issue link](https://github.com/DollhouseMCP/mcp-server/issues/1096)  
**Status:** Phase I Complete / Phase II In Progress

---

## Why I Chose This Issue

I chose this issue because it directly matches my strongest skill area — writing focused unit tests in TypeScript/Jest — while giving me a low-risk entry point into a real, actively maintained codebase (DollhouseMCP/mcp-server ships regular releases and has clear contribution docs). My learning goal is to get comfortable testing filesystem side effects and environment-variable mutation safely (temp directories, `HOME` overrides, singleton resets), a pattern I hadn't tested in isolation before. The issue itself is well-scoped: the maintainer already enumerated the exact functions and scenarios to cover, so I can validate my understanding of the existing `test-setup.ts` implementation before writing any test, rather than reverse-engineering requirements from a vague bug report. It's also unassigned with no open PR, so it's currently claimable.

---

## Understanding the Issue

### Problem Description

`test/__tests__/unit/portfolio/test-setup.ts` provides shared test-isolation utilities (`setupTestEnvironment`, `cleanupTestEnvironment`, `clearSuiteDirectory`, `resetSingletons`) that were introduced in [PR #1095](https://github.com/DollhouseMCP/mcp-server/pull/1095) to fix file-system pollution between test runs, but the utilities themselves have no dedicated unit tests. That matters because these helpers manipulate global, hard-to-observe state — temp directories and the `HOME` environment variable — so a silent regression here (e.g., `HOME` not restored, or a suite directory deleted mid-run) could reintroduce the exact cross-test pollution bug #1095 was meant to fix, and the failure would likely surface as a flaky, hard-to-diagnose test elsewhere in the suite rather than in this file. I chose to work on it because it's a self-contained, well-defined testing task that lets me practice mocking `fs`/`os`/`process.env` without touching production application logic.

### Expected Behavior

Add unit tests covering: directory creation and structure (`.dollhouse/portfolio`), `HOME` override/restore, suite-directory reuse vs. fresh creation, cleanup behavior (`cleanupFiles` true/false), suite directories surviving individual test cleanup, graceful error handling during cleanup, `clearSuiteDirectory()` cache reset, and `resetSingletons()` including dynamic-import failure handling. See the [full acceptance checklist](#testing-strategy) below, adapted directly from the issue's suggested test coverage.

### Current Behavior

`test-setup.ts` has no dedicated test file, so none of the behaviors above are formally verified — regressions in directory handling or environment restoration would only be caught indirectly (or not at all) via failures in unrelated test suites.

### Affected Components

- `test/__tests__/unit/portfolio/test-setup.ts` — the module under test (no existing spec file for it)
- `setupTestEnvironment()` — temp directory creation, `.dollhouse/portfolio` structure, `HOME` override, suite-directory reuse logic
- `cleanupTestEnvironment()` — `HOME` restoration, conditional temp-directory removal, suite-directory protection during per-test cleanup
- `clearSuiteDirectory()` — suite-directory removal and cache reset
- `resetSingletons()` — singleton reset via dynamic import, including ESM import-failure handling
- Related: [PR #1095](https://github.com/DollhouseMCP/mcp-server/pull/1095) (original test-isolation fix these utilities implement)

### Acceptance Criteria (what "fixed" looks like)

- [ ] Every function/scenario listed in issue #1096 has at least one corresponding unit test
- [ ] Tests pass in isolation and when run in parallel with the rest of the suite (no shared-state leakage)
- [ ] `HOME` is verifiably restored to its original value after both success and error paths
- [ ] Suite-directory reuse vs. cleanup semantics are each covered by a dedicated test (not conflated)
- [ ] New tests run cleanly in CI with no flake across repeated runs

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
