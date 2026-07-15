# su26-ai301-contribution
# Contribution [#1265]: Replace raw Tailwind colors with semantic names

**Contribution Number:** 1265  
**Student:** Yifei Feng  
**Issue:** [GitHub issue link](https://github.com/SwitchbackTech/compass-calendar/issues/1265)  
**Status:** Phase I Complete / Phase II Complete / Phase III Complete / Phase IV Complete

---

## Why I Chose This Issue

I chose Issue #1265, "Replace raw Tailwind colors with semantic names," because it is a well-scoped frontend refactor that fits both my skill set and my learning goals. The issue is clearly described, the project appears actively maintained, and it offers a practical way to improve maintainability without introducing a large feature change. I also reviewed the repository guidance and the issue discussion to confirm that the task was appropriate for a first contribution.

This issue is a strong match for me because it involves frontend styling, TypeScript, and design-system consistency, which are areas I want to strengthen. I also saw that the project had a milestone and no existing PR or assignee for the issue, which made it look claimable and relevant for contribution.

I left a comment on the issue introducing myself and expressing interest, and I used the discussion to better understand the expected direction before beginning the work.

---

## Understanding the Issue

### Problem Description

The Compass codebase currently uses raw Tailwind color values in several styling-related modules instead of the semantic design tokens the project expects. This creates inconsistency with the established styling system and makes future updates harder because colors are defined in multiple places rather than through a centralized token layer.

### Expected Behavior

The expected outcome is that the raw color usage should be replaced with the project’s semantic color tokens where appropriate, preserving the current visual result while making the styling system more consistent and maintainable. In practice, this means the affected frontend files should reference the semantic token layer instead of hard-coded or direct raw color values.

### Current Behavior

The raw color value is still present in multiple places across the web styling modules:
1. It is defined in `packages/web/src/common/styles/colors.ts` as a shared color constant.
2. It is used directly in `packages/web/src/common/styles/theme.util.ts` for WORK priority styling.
3. It appears in `packages/web/src/common/styles/theme.ts` as part of the accent light gradient configuration.
4. It is also embedded in `packages/web/src/common/constants/toast.constants.ts` for toast progress gradients.

These usages show that the semantic token system is not being used consistently for this color.

### Affected Components

| File | Usage | Impact |
|------|-------|--------|
| `packages/web/src/common/styles/colors.ts` | Exports `blueGray100` constant | Definition point—blocking deprecation |
| `packages/web/src/common/styles/theme.util.ts` | WORK priority color (`const WORK = c.blueGray100`) and hover brightening | Affects event/task priority tag styling |
| `packages/web/src/common/styles/theme.ts` | Gradient end (`end: c.blueGray100`) | Affects theme object's `gradient.accentLight.end` value |
| `packages/web/src/common/constants/toast.constants.ts` | Toast progress gradient (`${c.blueGray100}`) | Affects visual appearance of in-flight toast notifications |
| Components using priority colors | Event list, week view, task sidebar | Displays WORK priority events with color |
| Components using toast | Session expiration, form submissions, notifications | Displays toast with progress bar |

---

## Reproduction Process

### Environment Setup

I set up the project locally using the repository’s frontend workflow and reviewed the relevant project instructions in `AGENTS.md` before making changes. One challenge was that the initial environment setup was not straightforward because the repository has a more complex full-stack structure than the specific frontend issue required. I resolved this by narrowing the workflow to the web package and using the documented local development approach for the frontend rather than trying to run the entire stack at once.

### Steps to Reproduce

1. **Locate raw color definition:**
   - Open `packages/web/src/common/styles/colors.ts`
   - Find line 11: `blueGray100: "hsl(196 45 78)"`
   - Observe this is a raw color constant outside the semantic theming system

2. **Trace usages to theme utilities:**
   - Open `packages/web/src/common/styles/theme.util.ts`
   - Find line 6: `const WORK = c.blueGray100;`
   - Find line 27: `[Priorities.WORK]: brighten(c.blueGray100)`
   - Observe these are used for WORK priority event styling

3. **Check theme object pollution:**
   - Open `packages/web/src/common/styles/theme.ts`
   - Find line 26: `end: c.blueGray100` in the `accentLight` gradient
   - Observe this passes a raw color into the theme object

4. **Examine toast hardcoding:**
   - Open `packages/web/src/common/constants/toast.constants.ts`
   - Find line 19: `linear-gradient(to right, ${c.blue100}, ${c.blueGray100})`
   - Observe the gradient uses raw color constants instead of CSS variables

5. **Verify semantic alternative exists:**
   - Open `packages/web/src/index.css`
   - Search for `--color-gradient-accent-light-end`
   - Find definition: `--color-gradient-accent-light-end: hsl(196 45 78);`
   - **Result:** Semantic token exists with identical HSL value to `blueGray100`

### Reproduction Evidence

- **Commit showing reproduction:** [https://github.com/fyf820/compass-calendar/tree/refactor/replace-raw-tailwind-colors-with-semantic-names]
- **Screenshots/logs:** [bun dev:web
$ cd packages/web &&bun run dev.ts
[compass] building...
[compass] dev server → http://localhost:9080
[rebuild] common\styles\colors.ts
[rebuild] auth\posthog
[rebuild] common\constants\toast.constants.ts
[rebuild] auth\compass\session\SessionProvider.tsx
[rebuild] ducks\events\slices\event.slice.ts
[rebuild] common\utils\shortcut\shortcut.util.tsx
[rebuild] views\Week\interaction\targeting\weekCalendarEventTargeting.ts
[rebuild] views\Forms\EventForm\MigrateForwardMenuButton.tsx
[rebuild] common\utils\storage\storage.types.ts
[rebuild] common\styles\colors.ts
[rebuild] auth\posthog]
- **My findings:** [  -  Semantic token `--color-gradient-accent-light-end` exists in `index.css` with identical HSL value
  -  No other files appear to depend on `blueGray100` export (safe to deprecate)
  -  Tailwind v4 CSS custom property mapping supports automatic class generation
  -  Theme object structure supports direct semantic references
  -  This is a backward-compatible refactoring (visual output unchanged)
    ]
- **Important:** After reviewing the contribution guidance, I confirmed that the repository expects contributors to follow its process carefully, including contributor approval expectations before opening a PR.
  
---

## Solution Approach

### Analysis

The issue is rooted in a design-system inconsistency rather than a single functional defect. The raw color constant `blueGray100` is defined and reused across several frontend styling modules, which bypasses the semantic token structure that the project is already using elsewhere. This makes the palette harder to maintain and creates a mismatch with the repository’s documented styling guidance.

**Root cause:** `blueGray100` was introduced in `colors.ts` before the semantic token layer (`index.css`) existed, and no lint rule enforced token usage afterward—so it was never migrated when the token system was added. This plan replaces its usages in `colors.ts`, `theme.util.ts`, `theme.ts`, and `toast.constants.ts`, and adds a lint rule (`no-raw-tailwind-colors.ts`) to prevent recurrence.

### Proposed Solution

Replace all 5 usages of the raw color `blueGray100` with references to the existing semantic token `--color-gradient-accent-light-end`. This approach:
- Maintains visual identity (identical HSL values)
- Consolidates color definitions into the centralized semantic token system
- Enables single-point updates for future design changes
- Aligns with Compass design system principles
- Prepares infrastructure for light/dark mode support
- Is backward-compatible—no API changes, no visual changes for end users

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]
The codebase uses raw color constant `blueGray100` (HSL `196 45 78`) in 5 places across the styling system:
1. Definition in `colors.ts`
2. WORK priority color in `theme.util.ts` (line 6: `const WORK = c.blueGray100;`)
3. Hover state for WORK priority in `theme.util.ts` (line 27: `brighten(c.blueGray100)`)
4. Gradient end in `theme.ts` (line 26: `end: c.blueGray100`)
5. Toast progress gradient in `toast.constants.ts` (line 19: `${c.blueGray100}`)

This violates the design system rule to use semantic tokens, creating maintenance burden and blocking design system improvements.


**Match:** [What similar patterns/solutions exist in the codebase?]
1. **Semantic token definition** ([packages/web/src/index.css](packages/web/src/index.css#L15-L49)):
   - CSS custom properties defined in `@theme` block
   - Pattern: `--color-{category}-{name}: hsl(...)`
   - Automatically mapped to Tailwind utilities in v4

2. **Gradient token already exists:**
   ```css
   --color-gradient-accent-light-end: hsl(196 45 78);  /* Exact match to blueGray100 */
   --color-gradient-accent-light-start: hsl(202 100 67);
   ```

3. **Theme object consumption:**([packages/web/src/index.css(packages/web/src/common/styles/theme.ts)):
  ```
  gradient: {
    accentLight: {
    start: c.blueGray100,  // ← Replace with semantic reference
    end: c.blueGray100     // ← Replace with semantic reference
    }
  }
  ```

4. **CSS variable usage pattern::**
-  In styled-components: var(--color-panel-shadow)
-  In Tailwind: shadow-[0_0_10px_var(--color-panel-shadow)]
-  In gradients: linear-gradient(to right, var(...), var(...))

5. **Priority color mapping pattern:**([packages/web/src/index.css(packages/web/src/common/styles/theme.util.ts))):
-  Maps Priorities.WORK to color constant
-  Applies brightening for hover states
-  Used by event/task components

**Plan:** [Step-by-step implementation plan]
1. Update the relevant styling modules so they reference semantic color tokens instead of the raw color constant where appropriate.
2. Preserve the existing visual behavior while moving the implementation toward the project’s centralized design-token system.
3. Review the affected files carefully to ensure the changes stay scoped to the issue and do not introduce unrelated formatting changes.
4. Verify the results by checking the relevant project guidance and testing the frontend workflow for regressions.

**Implement:** [Link to branch](https://github.com/fyf820/compass-calendar/tree/refactor/replace-raw-tailwind-colors-with-semantic-names)

**Review:** I reviewed the repository guidance and used the available project instructions to make sure the changes followed the expected contribution approach. I also checked the relevant styling guidance and focused the work on the files directly tied to the issue.

**Evaluate:** I will verify that the change stays consistent with the project’s styling rules and that the impacted frontend files continue to behave as expected after the refactor.

---

## Testing Strategy

### Unit Tests

No dedicated unit test was added for this refactor because the issue focused on styling consistency rather than a behavioral bug. The work was validated through targeted review of the affected modules and the existing frontend workflow.

### Integration Tests

No separate integration test was introduced for this change.

### Manual Testing

I ran the local web workflow to inspect the affected UI areas and confirm that the refactor did not introduce obvious regressions. I also reviewed the available project tests and noted that any failures I saw were unrelated to this change.

---

## Implementation Notes

### Week 1 Progress

I investigated the issue by tracing the raw color usage through the relevant styling files and confirming where the semantic token system already existed. I also reviewed the project guidance to ensure that the approach stayed aligned with the repository’s contribution expectations.

### Week 2 Progress

I worked through the implementation carefully and resolved the main challenge of identifying all of the places that were still referencing the raw color. I also handled a merge conflict in the styling file by reviewing the surrounding context and preserving the intended changes.

### Code Changes

- **Files modified:**
- bun.lock
- packages/scripts/src/lint/no-raw-tailwind-colors.ts
- packages/web/src/common/styles/colors.ts
- packages/web/src/components/Shortcuts/ShortcutHint.tsx
- packages/web/src/index.css
- packages/web/src/views/Day/components/AddTask/AddTaskActiveButton.tsx
- packages/web/src/views/Day/components/AddTask/AddTaskPreviewButton.tsx
- packages/web/src/views/Day/components/ContextMenu/BaseContextMenu.tsx
- packages/web/src/views/Day/components/Icons/TaskCircleIcon.tsx
- packages/web/src/views/Day/components/Shortcuts/ShortcutTip.tsx
- packages/web/src/views/Day/components/Task/Task.tsx
- packages/web/src/views/Day/components/TaskList/TaskList.tsx
- packages/web/src/views/Day/components/TaskList/TaskListHeader.tsx
- packages/web/src/views/Day/components/Toasts/MigrationToast/MigrationToast.tsx
- packages/web/src/views/Day/components/Toasts/UndoToast/UndoDeleteToast.tsx
  
- **Key commits:**
- [1](https://github.com/SwitchbackTech/compass-calendar/commit/d33f868bb03ef27940dcc78366e4f82537bcbe24)
- [2](https://github.com/SwitchbackTech/compass-calendar/commit/9d6aac2f4ba520eb8c15443bc15a38be880879d3)
- [3](https://github.com/SwitchbackTech/compass-calendar/commit/a8ab648fc1b4a2be5227d85db146ed73c4cb360d)
- [4](https://github.com/SwitchbackTech/compass-calendar/commit/7b61ad96541fd862aac73c46121b6f342ca24ce5)
- **Approach decisions:** 
This approach can fix related semantic names and meet the guide of the issue description. It also meets the CONTRUIBUTING.md and AGENTS.md requirements.

---

## Pull Request

**PR Link:** [GitHub PR URL](https://github.com/SwitchbackTech/compass-calendar/pull/1891)

**PR Description:**

## Description

Closes [#1265](https://github.com/SwitchbackTech/compass-calendar/issues/1265)

Compass is moving toward a hybrid Tailwind + styled-components theming
strategy, but raw Tailwind palette values (e.g. `blue-gray-100`,
`gray-700`) are still hard-coded across the web package instead of
routed through semantic tokens. That makes light/dark theming
inconsistent, forces the same color to be updated in multiple places
whenever the palette or branding changes, and leaves no guardrail to
stop the next raw-color usage from being added.

This PR replaces raw Tailwind palette color classes throughout the web package with semantic token classes from the design system and adds a linting check to help prevent future violations.


## Changes

- **`colors.ts`**: Fixed duplicate `blue100` key — renamed second entry
  to `blueGray100` (was `undefined` at runtime); fixed missing `/` alpha
  separator in `gray700` HSL value
- **`index.css`**: Added two new semantic tokens:
  `--compass-color-kbd-bg` (keycap/badge backgrounds) and
  `--compass-color-interactive-accent` (interactive element accent)
- **10 TSX files**: Replaced all raw classes (`bg-gray-700`, `text-gray-300`,
  `border-gray-600`, `bg-blue-200/*`, `bg-darkBlue-400`, `text-white-100`,
  etc.) with their semantic equivalents
- **`packages/scripts/src/lint/no-raw-tailwind-colors.ts`**: New lint
  script that errors on any remaining raw palette class usage in `.tsx`
  files, integrated with `bun lint`

## Type of Change

- [x] Refactor (non-breaking change that improves code maintainability)

## Testing

- [ ] `bun test:web` passes
- [ ] `bun type-check` passes
- [ ] `bun lint` passes (no raw color violations)
- [ ] Manually verified UI appearance is unchanged in the Day view task
  panel, shortcut hints, toast notifications, and context menus

## Checklist

- [x] No new raw Tailwind palette colors introduced
- [x] Semantic token names follow the `compass-color-*` prefix convention
- [x] Lint script added to prevent regression

🤖 Generated with [Claude Code](https://claude.com/claude-code)

**Maintainer Feedback:**
- **2026-06-30** ([PR #1891 comment](https://github.com/SwitchbackTech/compass-calendar/pull/1891)): Maintainer `tyler-dane` rejected the PR, citing the project's updated policy toward "driveby AI PRs": *"Thanks for taking the time. However, I'm going to have to reject this. Please see CONTRIBUTING.md."*
- **2026-06-30**: No follow-up commits were made in response—the rejection was based on the updated contributor policy itself (not the code), so there was nothing in the diff to revise. The PR was closed as rejected at commit [7b61ad9](https://github.com/SwitchbackTech/compass-calendar/commit/7b61ad96541fd862aac73c46121b6f342ca24ce5), the last commit pushed before the feedback.

**Status:** Rejected

---

## Learnings & Reflections

### Technical Skills Gained

I have learned the prompting skills about how to work with AI with a detailed documentation. The AGENTS.md helps me a lot to guide AI to solve issues, that is a great example.

### Challenges Overcome

This issue has a lot of related files because of the change of variable names. I used AI to help me find and fix them, then audit them. That help me save a lot of time because this project codebase is huge. Also, this project has a very detailed CONTRIBUTING.md, it is hard to follow everything and I used AI to help me check if I followed the requirements for each step.

### What I'd Do Differently Next Time

I'll read the CONTRUIBUTION/README first to make sure that I can work on this project.

---

## Resources Used

The project has a very detailed documentations, such as README.md, CONTRIBUTING.md, AGENTS.md, and agents skills documentation including CodeX, Cursor, and Claude. They are very helpful to fix issue and leran the processes.
