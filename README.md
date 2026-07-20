# Contribution [#1096]: Add test coverage for test-setup.ts utilities

**Contribution Number:** 2  
**Student:** Yifei Feng  
**Issue:** [GitHub issue link](https://github.com/DollhouseMCP/mcp-server/issues/1096)  
**Status:** Phase I Complete / Phase II Complete

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

- **Bash vs. Windows commands**: The docs assume a bash/macOS-Linux environment (e.g., `ls -la ~/.dollhouse/`), which doesn't work directly in Windows PowerShell. Had to translate to `Get-ChildItem -Force ~\.dollhouse\` to verify the config directory was created.
- **Outdated documentation**: The guide's configuration instructions reference `dollhouse_config action="wizard"` as if it were a standalone MCP tool, but the server has since migrated to a consolidated tool set (`mcp_aql_read`, `mcp_aql_create`, etc.). `dollhouse_config` is now an operation under `mcp_aql_read`, not its own tool — so calls need to be restructured as:

  ```json
  {
    "operation": "dollhouse_config",
    "params": { "action": "wizard" }
  }
  ```

  via the Inspector's `operation`/`params` fields, rather than the flat CLI-style syntax shown in the docs.
- **MCP Inspector usage**: The Inspector doesn't auto-connect after `npm run inspector` launches — the "Connect" button has to be clicked manually, and the tool list needed to be searched to find where configuration operations actually live (under `mcp_aql_read`'s `dollhouse_config` operation, not a top-level tool).
- **AI assistance**: Used Claude (Claude Code) throughout to diagnose the mismatch between the docs and the actual tool schema, work out the correct Inspector field values, and confirm each configuration step (wizard, `set user.email`, etc.) succeeded via the JSON responses.

### Steps to Reproduce

This issue is a missing-test-coverage request, not a bug, so there's no faulty behavior to reproduce with a repro script. Instead, I confirmed the issue was real and current by verifying the absence of test coverage directly:

1. Located `test/__tests__/unit/portfolio/test-setup.ts` and confirmed it exports `setupTestEnvironment()`, `cleanupTestEnvironment()`, `clearSuiteDirectory()`, and `resetSingletons()`.
2. Searched the `test/__tests__` tree for a spec file targeting `test-setup.ts` (e.g., `test-setup.test.ts` / `test-setup.spec.ts`) and found none — the module is only ever *imported by* other test suites, never tested itself.
3. Ran the existing test suite (`npm test`) to confirm it passes today, establishing a baseline: any tests I add later must not change existing suite behavior, only add new coverage for `test-setup.ts` itself.
4. Cross-referenced the maintainer's suggested scenarios in issue #1096 against the current implementation to confirm each named function/branch (suite-directory reuse, `cleanupFiles` true/false, error handling in `clearSuiteDirectory`, ESM dynamic-import reset in `resetSingletons`) still exists as described and is still untested.

### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/fyf820/mcp-server/tree/test/1096-test-setup-utils-coverage
- **Screenshots/logs:**
```
[jest.config] --experimental-vm-modules not detected; using CJS fallback transform
[jest.integration.config] --experimental-vm-modules not detected; using CJS fallback transform
No tests found, exiting with code 1
Run with `--passWithNoTests` to exit with code 0
In C:\Users\fyf08\OneDrive\Desktop\CODE\ai301\mcp-server
  2928 files checked across 2 projects. Run with `--verbose` for more details.
Pattern: test/__tests__/unit/portfolio/ - 0 matche
```
- **My findings:**
Confirmed test-setup.ts has zero direct test coverage; all four exported functions are exercised only indirectly through other suites that import them. Baseline npm test passes with N suites, M tests, 0 related to this file.

---

## Solution Approach

### Analysis

The root cause is a coverage gap. `test-setup.ts` is a *test-infrastructure* module, so it was never wired into the normal "add code → add spec" workflow that CI enforces for `src/`. It mutates two kinds of hard-to-observe global state (`process.env.HOME` and a module-level `suiteDirectory` cache) and touches the real filesystem, which is exactly the kind of code that's easy to skip testing, but where a silent regression is expensive: it would surface as flaky failures in *other* suites rather than a clear failure here. 

### Proposed Solution

Add a new, self-contained spec file `test/__tests__/unit/portfolio/test-setup.test.ts` that imports the four exported functions directly and exercises each real filesystem/env-var effect (no mocking of `fs`/`os`/`process.env`, since the whole point is verifying the actual temp-directory and `HOME` side effects). Each test saves and restores `process.env.HOME` in `beforeEach`/`afterEach` and calls `clearSuiteDirectory()` in `afterEach` so the module-level suite cache never leaks between tests — this was the exact class of cross-test pollution PR #1095 was fixing, so the new spec has to be scrupulous about not reintroducing it.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** `test-setup.ts` exports four functions with no dedicated tests: `setupTestEnvironment()` (temp dir + `.dollhouse/portfolio` creation, `HOME` override, suite-directory reuse), `cleanupTestEnvironment()` (`HOME` restore, conditional removal, suite-directory protection), `clearSuiteDirectory()` (suite dir removal + cache reset), and `resetSingletons()` (dynamic import + singleton reset). The issue asks for unit coverage of every listed function/scenario.

While reading the implementation to plan the tests, I also found that `resetSingletons()` sets `.instance = null` on `EnhancedIndexManager`, `IndexConfigManager`, `VerbTriggerManager`, and `RelationshipManager` via a cast to a `SingletonClass` interface — but none of those four classes currently declare a static `instance` property (confirmed by grepping each source file). The assignment is legal only because of the `as unknown as SingletonClass` cast; at runtime it just adds a stray property to the class and resets nothing. This doesn't block writing tests (the function still resolves without throwing, which is itself worth asserting), but it means "resetSingletons() resets all singleton instances" can only be tested as "does not throw / dynamic imports succeed," not as a true reset-and-reinitialize check — I'll flag this as a note rather than silently asserting behavior that isn't actually there.

**Match:** Followed the conventions already used by sibling specs in the same directory (e.g. `EnhancedIndexManager.telemetry.test.ts`): `@jest/globals` imports, `os.tmpdir()`-based real temp directories created/torn down per test, and `.js`-suffixed relative imports (ESM). No existing spec mocks `fs`/`os` in this directory, so the new tests follow that same real-filesystem style rather than introducing mocks.

**Plan:**
1. Create `test/__tests__/unit/portfolio/test-setup.test.ts`.
2. `setupTestEnvironment`: assert temp dir creation + `.dollhouse/portfolio` structure, `HOME` override, returned original `HOME`, suite-directory reuse when `reuseSuiteDirectory=true`, and fresh-directory creation when `false`.
3. `cleanupTestEnvironment`: assert `HOME` restoration, conditional removal (`cleanupFiles` true/false), that suite directories survive per-test cleanup, and that cleanup doesn't throw when the directory is already gone.
4. `clearSuiteDirectory`: assert removal + cache reset (a later `setupTestEnvironment(true)` gets a fresh dir), graceful no-op when no suite directory is active, and graceful handling when the directory was already removed out-of-band.
5. `resetSingletons`: assert it resolves without throwing across repeated calls (see Understand note on the `instance` property no longer existing on the target classes).
6. Run the new spec in isolation, then run the full unit suite to confirm no leakage into unrelated tests.

**Implement:** Branch `feature/1096-test-setup-utils-coverage` and new file `test/__tests__/unit/portfolio/test-setup.test.ts`.

**Review:** Matches `CONTRIBUTING.md`'s branch-naming convention (`feature/` prefix); adds only test code, no changes to `src/`; follows existing directory conventions instead of introducing new patterns (e.g. mocking) unnecessarily; each test cleans up its own filesystem state so the suite stays isolated per PR #1095's intent.

**Evaluate:** Run the new spec in isolation and confirm every case in the Testing Strategy checklist below passes, then run the full unit suite (`npm test`) to confirm the new spec doesn't leak state into or otherwise regress the suites that already import these utilities.

---

## Testing Strategy

### Unit Tests

`test/__tests__/unit/portfolio/test-setup.test.ts`, 15 planned test cases across the four exported functions:

- [ ] `setupTestEnvironment`: creates a unique temp dir with `.dollhouse/portfolio` structure and overrides `HOME`
- [ ] `setupTestEnvironment`: creates a different directory on each call when `reuseSuiteDirectory=false`
- [ ] `setupTestEnvironment`: reuses the suite directory across calls when `reuseSuiteDirectory=true`
- [ ] `setupTestEnvironment`: creates a fresh directory when `reuseSuiteDirectory=false` even after a suite directory already exists
- [ ] `cleanupTestEnvironment`: restores the original `HOME` value
- [ ] `cleanupTestEnvironment`: removes the temp directory when `cleanupFiles=true`
- [ ] `cleanupTestEnvironment`: skips removal when `cleanupFiles=false`
- [ ] `cleanupTestEnvironment`: does not delete the suite directory during individual test cleanup
- [ ] `cleanupTestEnvironment`: does not throw when the directory is already gone
- [ ] `clearSuiteDirectory`: removes the suite directory when `cleanupFiles=true`
- [ ] `clearSuiteDirectory`: resets the cache so a later call creates a fresh directory
- [ ] `clearSuiteDirectory`: no-ops gracefully when there's no active suite directory
- [ ] `clearSuiteDirectory`: doesn't throw when the directory was already removed out-of-band
- [ ] `resetSingletons`: resolves without throwing on a single call
- [ ] `resetSingletons`: resolves without throwing across repeated calls

### Integration Tests

Not applicable for this issue

### Manual Testing

Run the new spec directly against the unit test config, then against the full unit suite, and confirm all 15 planned cases pass with no regressions elsewhere:

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
