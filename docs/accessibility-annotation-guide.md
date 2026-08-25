# Accessibility Annotation Guide

For designers annotating mockups for accessibility, to WCAG AA. This describes the **requirement**, not the implementation: no ARIA attributes, no iOS/Android APIs. That part is for developers.

This reference covers annotation meaning and content. When drawing annotations in Figma, also follow `accessibility-annotation-presentation.md` for the shared tooltip component, arrow routing, and target-placement rules.

## TL;DR

Annotate only what is ambiguous or missing. Everything else is assumed decorative or read natively.

## What to annotate vs skip

**Less is more.** Every annotation is a decision someone must act on. Only flag what is ambiguous, missing, or unique to this screen.

The most common example of this rule: **icons and images are decorative by default.** Only annotate one if it carries meaning the layout doesn't already convey (a chart, a photo used as evidence, an icon that's the only cue for an action).

| Annotate | Skip |
|---|---|
| Custom or ambiguous interactive roles (is this a button or a link?) | Plain body text, labels, captions |
| Meaningful images or icons that need alternative text | Decorative shapes, backgrounds, confirmed-decorative icons |
| Section headings and their level | Anything already covered by an existing component spec |
| State not shown by color alone (selected, expanded, disabled, error) | Redundant text that just repeats the visible label |
| Focus or reading order surprises | Layout containers with no interactive role |
| Content that updates without navigation | Anything already flagged elsewhere |

## Annotation format

Use four fields. Most annotations only need one.

| Field | When to use | Good example | Avoid |
|---|---|---|---|
| Output | What should be announced, when it differs from the visible text | `Output: Set passengers. Current value: 3 passengers` | ~~`Output: Add passengers, click to update, button`~~ (restates the role) |
| Role | The control's function isn't visually obvious | `Role: Link` | ~~`Role: Clickable text`~~ (not a real role) |
| State | A state must be perceivable, not just visual | `State: Selected` | ~~`State: Blue background`~~ (describes appearance, not state) |
| Requirement | Anything else: a decision, a gap, a behavior | `Requirement: Announce when this value updates without navigation` | ~~`Requirement: Use aria-live`~~ (implementation detail) |

## Alternative text

An alternative text is a screen reader's substitute for what is visually presented. Not every element needs one: if there is visible text, in most cases no alternative text should be provided. There are exceptions.

- Describe function or content, not appearance: `"Trip confirmed"`, not `"Image of a hand"`.
- Skip filler text like "image of" or "icon of".
- Complex elements (like `Item` components with "data" suffixes) benefit from a single, unified alternative text. This prevents screen readers stopping at each line of content: `"Total price for one passenger is nine euros and ninety nine cents"`, not `"Total price"` / `"One passenger"` / `"Nine euros ninety nine cents"`. Same idea as the `Output` field above.
- Don't annotate decorative images: the absence of alternative text already means decorative.

## State communication

Never rely on color alone. If a state changes an element's meaning, name the state directly.

`State: Expanded`, not "the chevron rotates".

## Roles reference (appendix)

Look these up when a control's role isn't obvious. Not meant to be read top to bottom.

| Pattern | Expected role |
|---|---|
| Navigates to another screen or page | Link |
| Triggers an action on the same screen | Button |
| One of a set of mutually exclusive views | Tab (inside a Tablist) |
| Binary on/off setting | Switch |
| Single choice from a list | Radio button |
| Multiple independent choices | Checkbox |
| Related radio buttons or checkboxes | Grouped field (one shared group name, not just individual labels) |
| Selecting a value in a range | Slider |
| Increment/decrement counter | Stepper |
| Single-line text entry | Text field |
| Multi-line text entry | Text area |
| Search field | Search field |
| Dropdown, choosing from a closed list | Select |
| Typeahead or autocomplete field | Combobox |
| Temporary overlay blocking the rest of the screen | Dialog |
| Content that updates without a page change | Live region |
| Inline success, error, or warning message | Alert |
| Loading or progress indicator | Progress indicator |
| List of links showing current location (e.g. Home > Trips > Booking) | Breadcrumb |
| Expandable section header | Disclosure (button plus the region it controls) |
| Popover with extra info shown on hover or focus | Tooltip |
| List of actions revealed from a trigger | Menu |
| Section title | Heading (h1 to h6, matching hierarchy) |
| Image that carries meaning | Image with alternative text |
| Purely visual image | Decorative image |
