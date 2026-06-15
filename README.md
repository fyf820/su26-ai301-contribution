# su26-ai301-contribution
# Contribution [#1265]: [Replace raw Tailwind colors with semantic names]

**Contribution Number:** [1265]  
**Student:** [Yifei Feng]  
**Issue:** [[GitHub issue link](https://github.com/SwitchbackTech/compass-calendar/issues/1265)]  
**Status:** [Phase I x / Phase II / Phase III / Phase IV] [In Progress x / Complete]

---

## Why I Chose This Issue

I choosed Issue #12345 "Replace raw Tailwind colors with semantic names" because it is a simple issue and has a detailed issue description listed how to solve this issue. It is labeled as "good first issue".
I'm interested in this because:
1. It is a easy issue for beginner to learn how the open source contribution works and I do not need much time to study tech stacks.
2. The related tech stack is frontend, and I have previous experience that can help me handle potential issue.
3. The maintainer add "Milestone: Jan-Mar 2026" that shows they are actively maintaining this project, and there is no PR or assigned developer.

From reading the issue thread, I understand the current problem is that  all instances of the raw Tailwind color color-blue-gray-100 need to be replaced  with an appropriate semantic color variable throughout the codebase. My contribution will improve the code maintainability and readability.

Left a comment on the issue introducing myself, but the maintainer doesn't confirme it's 
still open and pointed me toward the relevant code yet.

---

## Understanding the Issue

### Problem Description

[The Compass codebase currently uses the raw Tailwind color in multiple places across the styling system. This violates the design system principle documented in `.cursorrules/styling.md` which mandate that all colors must use semantic tokens rather than raw color constants.]

### Expected Behavior

[All usages of the raw color should be replaced with the existing semantic token like `--color-gradient-accent-light-end` (which has an identical HSL value). This maintains visual identity while bringing the code into alignment with the design system standards.]

### Current Behavior

[The raw color constant is:
1. Defined in `packages/web/src/common/styles/colors.ts` as a JavaScript constant, but some of the color variable names have been ficed.
2. Imported and used directly in `theme.util.ts` for priority color mapping (WORK priority)
3. Referenced in `theme.ts` as the gradient end color in the accent light gradient
4. Embedded in toast progress bar gradient construction in `toast.constants.ts`
]

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

My SDK conflicted with some dependencies, so I asked Copilot to help set up and configure the development environment. However, when running the app (the issue was specific to the frontend), the application behaved differently compared to the deployed production version. I cannot check relevent component in the running app because of AUTH issue. I guess this happened because the AI ​​attempted to run a full-stack environment, whereas the project itself was incomplete and the deployed production version is different with the develpment version. I later found the relevant documentation in `AGENTS.md` and switched to running the frontend separately in the terminal.

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
- **Important:** When I check CONTRIBUTING.md, I realized that my PR would not be approved by the project maintainers, because the project requires developer to apply and be approved as a contributor before being permitted to submit a PR.
  
---

## Solution Approach

### Analysis

The raw color constant `blueGray100` is defined and exported from `packages/web/src/common/styles/colors.ts`, then imported directly into styling utilities and theme configuration instead of using the semantic theming system. This creates a direct dependency that:
1. Bypasses the semantic token system defined in `index.css` with `@theme` directive
2. Prevents centralized color management and easy palette updates
3. Complicates future light/dark mode support
4. Violates the established design system rule: "Do NOT use raw colors - use semantic colors instead" (documented in `.cursorrules/styling.md` and tracked in issue #1265)

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
1. Update theme.ts — Replace raw color with semantic reference

Change line 26: end: c.blueGray100 → end: c.gradientAccentLightEnd (or extract from theme object once defined)
Alternative: directly use hsl(196 45 78) inline or reference CSS var
Verify theme.color.gradient.accentLight.end resolves correctly

2. Update theme.util.ts — Replace WORK priority color

Change line 6: const WORK = c.blueGray100; → Use semantic color from theme object
Change line 27: brighten(c.blueGray100) → Use brightened semantic color
Option: Map to theme.color.status.info or theme.color.tag.one (sky blue) since WORK priority is sky blue
Ensure hover state brightness matches current behavior

3. Update toast.constants.ts — Replace gradient with CSS variables

Change line 19: linear-gradient(to right, ${c.blue100}, ${c.blueGray100}) → Use CSS variables
New: linear-gradient(to right, var(--color-gradient-accent-light-start), var(--color-gradient-accent-light-end))
Verify gradient renders identically in toast notifications

4. Update colors.ts — Deprecate raw color

Add deprecation comment above blueGray100 export
Comment: // @deprecated Use --color-gradient-accent-light-end from semantic tokens instead. See issue #1265.
Do NOT remove yet—keep as fallback for other potential usages

5. Type safety & theme object (conditional):

If theme.ts needs new exports, add gradientAccentLightEnd to DefaultTheme type
Or reference existing gradient tokens consistently

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
