---
## 📄 OVERVIEW PAGE
---

## Chip basics

- Use Chip when users need to quickly filter results, switch between views, or trigger a lightweight action without leaving the current page.
- Use Chip to make state visible at a glance, such as active filters, active section, or active mode.
- Choose Chip over Tag when interaction is required; Tag is read-only labeling.
- Keep Chip behavior predictable by using one chip type per row: filter, navigation, or action.
- Chip supports three usage variants: Filter chip, Navigation chip, and Action chip.

![Filter chip](placeholder-filter-chip.png)

**Filter chip**

Use when users need to narrow a result set and keep selected criteria visible until removed.

![Navigation chip](placeholder-navigation-chip.png)

**Navigation chip**

Use when chips act like tabs to switch between peer sections in place.

![Action chip](placeholder-action-chip.png)

**Action chip**

Use for quick, grouped, comparable actions where the result is immediate.

## Quick links

- [Figma component doc](https://www.figma.com/design/dNrv9HF101jIFC8SRBufbf/Candidate-components?node-id=6009-1127&t=0CN6H8trGoe19MiG-4)

---
## 📄 USAGE PAGE
---

## When to use?

- Use Chip when options should stay visible and tappable in context, so users do not need to open extra menus for common tasks.
- Use Chip when quick selection or switching improves speed and clarity, especially above lists, maps, or sectioned views.

### Filter chip

![Filter chip usage](placeholder-filter-chip-usage.png)

- Choose Filter chip when users apply one or more visible criteria to refine a list.
- BlaBlaCar example: Search results quick filters above the ride list.
- Keep selected filters visibly on until users remove them, so current constraints remain obvious.
- Prefer dropdowns or list filters when options are numerous or need search.

### Navigation chip

![Navigation chip usage](placeholder-navigation-chip-usage.png)

- Choose Navigation chip when users switch between sibling destinations in the same page context.
- BlaBlaCar example: Your rides tabs such as Upcoming and Past.
- Use this pattern when there are more than two destinations or when the set can grow.
- Prefer segmented control for a strict two-state single-select toggle.

### Action chip

![Action chip usage](placeholder-action-chip-usage.png)

- Choose Action chip for one-tap, reversible, low-risk actions in a grouped control area.
- BlaBlaCar example: Map my search actions for swapping departure and arrival and opening filters.
- Use a verb-led label so the action is clear before tap.
- Prefer Button for a single standalone primary action.

## Minimal content rules

![Minimal chip content](placeholder-minimal-chip.png)

A chip must contain a short label; icons are optional and should support recognition or state, not decoration.

## Do's and Don'ts

### ✅ Do

- Keep chip rows semantically consistent so users can predict what tapping will do.
- Provide clear selected state for filter chips and persist it until removed.
- Add a clear-all affordance when multiple filters can be active.
- Use leading icons only when they clarify meaning or state.

### ❌ Don't

- Don't use a filter chip for a binary on/off setting; use Switch instead.
- Don't mix filter, navigation, and action chips in one row.
- Don't use navigation chips that do not match real page sections.
- Don't trigger destructive or irreversible actions from action chips without confirmation.
- Don't show disabled chips without an obvious reason.

---
## 📄 CONTENT PAGE
---

## Writing guidelines

Chip copy should be scannable, compact, and immediately understandable in context.

### Tone

- Be direct and functional.
- Favor clarity over personality.
- Keep labels neutral unless the action itself implies urgency.

### Length

- Keep labels to 1-3 words.
- Avoid punctuation in labels.
- Use sentence case.
- Truncate with ellipsis after the product max character limit.
- Never wrap onto multiple lines.

### Structure

**Filter chip label**
- State the value or condition directly.
- Avoid verbs.
- Keep terms recognizable from filter context.

**Navigation chip label**
- Match destination names exactly.
- Keep labels stable over time to preserve user memory.
- Avoid dynamic wording that changes meaning between sessions.

**Action chip label**
- Use a verb or clear action phrase.
- Describe what happens immediately after tap.
- Keep outcome obvious without needing helper text.

**Icons**
- Use leading icons only.
- Use icons for recognition or state, such as swap or selected checkmark.
- Never add decorative-only icons.

## Examples

### ✅ Good examples

| Context | Text |
|---------|------|
| Search results filter | Direct only |
| Search results filter | Under €20 |
| Your rides navigation | Upcoming |
| Your rides navigation | Past |
| Map my search action | Filters |
| Map my search action | Swap |
| Map my search action | Departure ⇄ Arrival |

### ❌ Avoid

| Instead of... | Try... | Why? |
|---------------|--------|------|
| Show me direct rides only! | Direct only | Removes verbosity and punctuation for faster scanning. |
| Tap to see upcoming rides | Upcoming | Navigation labels should match destination names. |
| Settings | Filters | Action labels should describe the actual immediate action in context. |
| Fast and cheap and comfortable | Fast… | Long labels reduce scan speed and break layout consistency. |

## Localization

- Reserve room for expansion in translated labels while preserving single-line behavior.
- Re-validate truncation in longer locales so meaning remains clear.
- Avoid ambiguous abbreviations that may not translate consistently.

---
## 📄 ACCESSIBILITY PAGE
---

## Screen reader experience

Screen reader users should hear Chip as a clear control with role and state that match behavior.

### Announcement

When a screen reader user encounters Chip, they hear role plus label, and when relevant, selected state.

> "Direct only, toggle button, selected"

> "Upcoming, tab, selected"

> "Swap, button"

Use semantics that reflect intent:
- Filter chip: toggle control with selected or not selected state.
- Navigation chip: tab-like control with selected state for active destination.
- Action chip: standard button with no persistent selected state unless explicitly modeled as toggle.

### Interaction

- Focus: Chip enters the normal focus order.
- Activation: Enter and Space activate the chip on web; equivalent activation gestures apply on mobile assistive tech.
- Feedback: Users receive immediate state or context feedback, such as selected state updates, content refresh, or modal open.

## Keyboard navigation

| Key | Action |
|-----|--------|
| Tab | Move focus to next chip/control |
| Shift+Tab | Move focus to previous chip/control |
| Enter | Activate focused chip |
| Space | Activate focused chip |
| Arrow keys | Move between chips when implemented as a tablist/navigation group |

## Design considerations

- Keep behavior consistent within a chip row so role expectations stay stable.
- Ensure selected state is perceivable without relying on color alone.
- Provide visible focus indicators on all interactive chips.
- Explain disabled state with nearby helper text or reason when possible.
- Include clear-all controls when multiple filter chips can be selected to reduce interaction burden.

### Color and contrast

- Ensure text and key state indicators meet WCAG AA contrast requirements.
- Don't encode meaning with color alone; pair warning or selected visuals with explicit text or icon state.

### Motion

Chip state transitions should be subtle and must not be required to understand state changes.

- Keep transitions brief and non-essential.
- Respect reduced-motion preferences where supported.
