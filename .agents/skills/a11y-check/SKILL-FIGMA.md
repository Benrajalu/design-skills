---
name: a11y-check
description: Produce minimal, actionable accessibility annotations for a single feature screen in Figma. Places tooltip annotations beside the mockup and delivers a written summary of designer actions. Use when asked to annotate, audit, or review a Figma screen for accessibility.
compatibility: Requires Figma MCP (mcp_figma_* tools)
---

# A11y Check - Skill

> **Purpose:** Produce a minimal, actionable set of accessibility annotations for a single feature screen. Place tooltip annotations beside the Figma mockup and deliver a written summary of designer actions not covered by tooltips.
>
> **What this skill does NOT do:** audit the full design system. It focuses on the *feature layer* - the specific arrangement, content, and context of elements on this screen.

---

## Single-file execution contract

This `SKILL-FIGMA.md` is intentionally self-contained for Figma import.

Use it in two layers:
1. **Core runtime contract (authoritative)**: everything up to and including `What this skill does NOT annotate`.
2. **Appendix A implementation details**: `Annotation Drawing Reference` and its subsections.

Execution rule:
- Use the core runtime contract first.
- Consult Appendix A only for Stage 7 drawing mechanics or if placement output is wrong.

---

## Design principle: minimal annotations

The goal is the **fewest annotations that carry real value**. Every tooltip is a decision the designer must act on. Annotation fatigue defeats the purpose.

Only annotate what is ambiguous, missing, or bespoke to this screen. Skip anything handled natively by assistive technology or already fully covered by a component spec.

Icons and images are decorative by default: only annotate one if it carries meaning the layout doesn't already convey. This is the most common application of the rule above.

## Conversation style

Keep chat with the designer brief and decision-focused. Do not narrate internal analysis, MCP calls, layer-tree observations, WCAG theory, or platform background unless the user asks or a decision depends on it.

Use one-line progress updates only at stage boundaries or when blocked.

Ask only necessary questions, and keep each question to one short sentence plus a recommendation.

Preferred style example: `Calendar icon: decorative? My suggestion: yes.`

The em dash character is prohibited everywhere in this skill's output, including chat messages, proposed plans, summaries, tooltip titles, and tooltip content. Use periods, commas, colons, parentheses, or hyphens instead.

## Platform scope

Annotate for **web (WCAG AA), native iOS, and native Android** in every run. Annotations describe the accessibility **requirement** - not the implementation. Do not include platform-specific syntax, APIs, or code in tooltip content. Leave implementation to developers.

- ✅ `"Value must be announced with its unit"` - platform-agnostic
- ❌ `"Use aria-label='87 points'"` - web-only implementation

---

## Scope: one frame per run

Run this skill once per Figma frame. For multi-screen flows, run once per screen - containers are named after the frame and coexist cleanly.

---

## Stage 1 - Acquire the frame

Require a Figma URL with a `node-id` before proceeding:

```
https://www.figma.com/design/{fileKey}/{fileName}?node-id={nodeId}
```

If the URL points to a page root (no `node-id`), ask the user to select a specific frame. If the user provides multiple screens, ask them to pick one.

---

## Stage 1b - Verify MCP availability

Run both probes **in parallel** immediately after extracting `fileKey` and `nodeId`. Use the lightest call available for each MCP.

| Probe | Call | Reused in |
|-------|------|-----------|
| Figma | `get_metadata(fileKey, nodeId)` | Stage 2 - do **not** call again |
| Supernova | `get_documentation_page_list()` | Stage 4 - do **not** call again |

| Result | Action |
|--------|--------|
| Figma unavailable (tool not found, auth error, timeout) | **Hard stop.** Tell the user the Figma MCP is not reachable and do not proceed. |
| Supernova unavailable | **Soft fail.** Warn the user once, then continue - activate the fallback path in Stage 4. |
| Both succeed | Cache both results and continue. |

**Figma hard-stop message:**
> "I can't reach the Figma MCP (error: `[error text]`). Check that the MCP server is connected and authenticated - see the Setup section of the README - then start a new conversation."

**Supernova soft-fail message:**
> "Supernova is not reachable right now. Component coverage checks will be skipped - I'll ask you directly about any spec gaps as we go."

---

## Stage 2 - Gather context

`get_metadata` is already available from Stage 1b - **do not call it again**. Only call `get_design_context`:

```
get_design_context(fileKey, nodeId, disableCodeConnect: true)
```

`get_design_context` returns visual structure, component names, fills, and CSS padding.  
`get_metadata` (from Stage 1b) returns the layer tree with node IDs, types, names, and bounding boxes.

Record the frame's canvas bounding box:

```javascript
const FRAME_LEFT   = frame.x;
const FRAME_TOP    = frame.y;
const FRAME_RIGHT  = frame.x + frame.width;
const FRAME_BOTTOM = frame.y + frame.height;
```

---

## Stage 3 - Analyse for WCAG AA requirements

Walk the layer tree and ask: **"What would need to be done to make this screen accessible?"** across web (WCAG AA), native iOS, and native Android.

For each element, identify concerns - do not filter yet. Common areas to check:

- Interactive elements: is the role clear? Is the name derivable?
- Form fields: are labels programmatically associated?
- Imagery: meaningful or decorative?
- State: communicated beyond colour alone?
- Focus/reading order: logical for the layout?
- Live regions: anything that updates without navigation?
- Component roles: custom controls that need explicit role declaration?

Touch target size is not a tooltip concern: it is directly measurable from `get_metadata` bounding boxes, not a judgment call. Check it separately and report undersized targets in Stage 8.

Produce an internal list of **concerns**, each with: element, concern type, and sparse tooltip fields:

| Field | Use when |
|-------|----------|
| `output` | The accessible name or announced output differs from, combines, or adds to visible text. Include current values here when they are part of what should be announced, e.g. `Set passengers. Current value: 3 passengers`. Omit when visible text already provides the correct accessible name. |
| `role` | The element is suspected to need semantic role confirmation, especially interactive elements or controls where role may be missing, custom, or context-dependent. |
| `state` | A programmatic state must be exposed: selected, expanded, checked, on/off, disabled, invalid, pressed, current page, etc. |
| `requirement` | A design decision, implementation gap, or dynamic behavior requirement remains after output, role, and state are clear. |

Do not produce full screen reader scripts by default. Tooltips should capture the missing or risky accessibility decision, while skipping fields that are already obvious or covered by the component spec. When in doubt about what to write or which tooltips to keep, let the designer arbitrate.

### Hard skips (do not raise as concerns)

| Condition | Reason |
|-----------|--------|
| Plain body text, labels, captions (`TEXT` node with no heading role) | Read natively by screen readers - no annotation needed |
| Decorative shape / background fill with no image | No semantic content |
| Layout container (`FRAME`/`GROUP`) with no interactive role and no image fill | Pure layout |
| Illustration or Icon confirmed as decorative | Decorative is the default; nothing actionable |
| Component whose entire visible output is readable text (body copy, labels) | Text is announced natively - check via `get_design_context` before raising |
| Anything the user has explicitly said to skip | User has waived it |

> **Do NOT skip section headings.** A text element that visually functions as a section heading (larger or bolder than surrounding text, or whose layer name contains "Heading", "Title", or "H1"-"H6") requires explicit heading role markup. Annotate it with the `"Heading"` annotation type and suggest a heading level from `h1` to `h6` when inferable from the screen hierarchy. Ask the designer when the level is ambiguous.

### Batching repeated concerns

When the same requirement applies to **multiple instances of the same component** on the screen, use a single **catch-all annotation** on one representative instance rather than annotating each row individually.

**When to batch:**
- 3 or more instances of the same component share an identical concern
- The requirement is uniform (same label strategy, same role, same pattern) - no instance-specific decision is needed
- The annotation does not include instance-specific `output` or `state`, unless every batched instance shares the same value

**How to annotate:**
- Pick the first or most visually prominent instance as the target
- Word the requirement to make the scope explicit: *"Applies to all [Component] rows on this screen"* or *"Applies to all Items with supplementary text in this list"*
- Keep catch-all wording generic. Do not use a specific label or value, such as `Label 'Destination'`, when the tooltip applies to multiple fields.
- Add any instance that has a **unique, screen-specific decision** (e.g. an unusual label combination, an inline action) as a **separate annotation** alongside the catch-all

**Example:** A settings screen with 8 Item/Navigation rows, all needing unified accessible labels → one catch-all annotation on the first row, plus a second annotation on the "Home address" row that also has an inline "Verify" action.

**Target mismatch prevention:** A tooltip arrow is a contract. Every populated field must be true for the element at the arrow endpoint. If `output` or `state` belongs to a different element, move the arrow, split the annotation, or rewrite it as a generic catch-all.

---

## Stage 4 - Component coverage check

For each concern that involves a named component instance:

1. **Batch lookup**: collect all component names, then call `get_design_system_component_list`. Use the `get_documentation_page_list` result already cached from Stage 1b - do **not** call it again.
2. For each match, call `get_documentation_page_content` and **read the Accessibility section in full**.
3. Judge coverage against the specific concern:

| Coverage result | Action |
|----------------|--------|
| Spec fully addresses this concern for this feature usage, **and** the correct answer is unambiguous in this context | **Silent skip** - remove from concern list, no annotation, no user question |
| Spec covers the requirement but the **correct value depends on context** (e.g. role varies by usage, label is screen-specific) | **Keep** - surface as a Stage 5 question |
| Spec partially addresses it or leaves a feature-specific decision open | **Keep** - annotate the gap |
| No spec, spec is empty, or spec is too generic | **Keep** - treat as `needs-decision` |

Do not assume a spec exists or is sufficient. Read it. A spec that says "follow platform accessibility guidelines" without specifics is too generic.

**Context-dependent role example:** A spec that says *"Item/Navigation must have `role="link"` or `role="button"`"* covers the requirement - but which role is correct on *this screen* depends on whether the item navigates to another page (link) or triggers an in-page action (button). That choice is screen-specific and must be confirmed with the user.

**No Supernova redirect tooltips.** Annotations must be self-contained and immediately actionable.

### When Supernova is unavailable

Supernova unavailability was detected and reported in Stage 1b - do not repeat the warning. For each concern that would normally trigger a component lookup, ask the user directly:

> "For **[Component Name]** - does it have an accessibility spec that covers [specific concern]? If yes, tell me what it says. If no, tell me what the requirement should be."

---

## Stage 5 - Clarifying questions

After stages 3–4, ask the user about anything that is still ambiguous. Ask **one question at a time** and wait for the answer before asking the next. Combine similar questions when the same answer applies to several elements.

**Question order:**
1. Ambiguous imagery / illustrations (decorative vs meaningful)
2. Standalone icons (redundant vs meaningful)
3. Interactive groups (single focusable unit vs individually traversable)
4. Custom components whose role is unclear
5. **Context-dependent roles** - for any component where the spec lists multiple valid roles (e.g. button vs link), ask which applies on this screen. Offer a catch-all if all instances behave the same way.
6. **Output and state composition** - when a visible label, value, or state must be announced together, ask whether it belongs in `output`, `state`, or `requirement`. Prefer `output` for natural announced values such as `Set passengers. Current value: 3 passengers`.
7. Focus order (one question for the whole screen, ask last)

### Question format

> **[Layer name]:** [decision needed]. My suggestion: [concrete recommendation].

Use the layer name from `get_metadata`. Keep suggestions concrete so the user can just say "yes" or give a one-line correction. Do not add background paragraphs.

Accept "out of scope", "predates this feature", or "covered elsewhere" without asking for justification - skip the element and move on.

---

## Stage 6 - Plan

After all questions are resolved, present the proposed annotation plan **before drawing anything**.

Format:

```
## Proposed annotations - [Frame name]

| # | Element | Annotation | Why |
|---|---------|-----------|-----|
| 1 | Passenger selector | Output: Set passengers. Current value: 3 passengers | Visible label and value must be announced together. |
| 2 | Search button | Role: Submit button | Primary form action. Semantic role should be explicit. |
| 3 | From / To inputs | Requirement: Labels must be programmatically associated | Visible labels may not be associated in implementation. |
| 4 | Terms row | Role: Link | Role depends on whether it navigates or performs an action. |

**Not annotated (and why):**
- Banner: decorative image - no alt text required
- Illustration (empty state): decorative - default behaviour
- MenuBar: out of scope (your request)

**Designer actions not covered by tooltips:**
- [list anything that requires design decisions but can't fit a tooltip]
```

Then ask: **"Does this plan look right?"**

Do not proceed to Stage 7 until the user confirms the plan.

Keep the plan compact. Use the table as the main communication. Keep `Why` entries short, ideally 3-8 words. In `Not annotated`, include only surprising skips, user-requested skips, or likely objections. Do not list every decorative, native-readable, or spec-covered element.

Before drawing, run this internal self-check for every annotation: **Does the arrow target own every populated field?** If not, retarget, split, or rewrite as a catch-all.

---

## Stage 7 - Place annotations

For each confirmed annotation, finalise:

| Field | Content |
|-------|---------|
| `label` | Short target label naming what the arrow points to (e.g. `"Passenger selector"`, `"Search button"`, `"Travellers field"`, `"Recent searches heading"`) |
| `output` | Optional. Accessible name or announced output when it differs from, combines, or adds to visible text |
| `role` | Optional. Semantic role when it needs confirmation or is likely to be missing |
| `state` | Optional. Exposed state when relevant |
| `requirement` | Optional. Specific, platform-agnostic requirement not already covered by output, role, or state |
| `side` | Tooltip side (`right`, `left`, `top`, `bottom`) |
| `elementEdgeX/Y` | Canvas coordinate of the **target element's own** edge facing the tooltip - not its container's edge, not the frame edge. Read from `get_metadata` for top-level nodes; derive from padding analysis for sub-elements. See [Determining element coordinates](#determining-element-coordinates). |
| `elementCenterX/Y` | Canvas centre of the element on the axis perpendicular to the arrow |

Use `label` as the tooltip title. It must identify the target element, not repeat the body field type. Do not use generic titles such as `Role`, `Output`, `State`, `Input`, or `Submit` unless that is actually the target's visible name.

Build the tooltip content from only the populated fields, one per line, in this order: `Output`, `Role`, `State`, `Requirement`. Do not include empty field labels. Do not include activation hints such as `click`, `tap`, or `press` in `output` when `role` already conveys interaction.

Every populated field must describe the exact target at the arrow endpoint. Catch-all annotations may describe shared role or requirement patterns, but must not include instance-specific `output` or `state` unless all targets share the same value.

For tab groups:
- Point to the selected tab when using `State: Selected`.
- Point to the matching tab when using tab-specific `output`, such as `Output: Stays, new`.
- Point to the tab group container for `Role: Tablist` or generic requirements that apply to all tabs.

For form fields:
- Point to a specific field when using a specific label or value, such as `Output: Travellers. Current value: 1 traveller`.
- For all-field catch-alls, use generic wording, such as `Requirement: Labels must be programmatically associated. Applies to all search form fields.`

Then follow the drawing reference below exactly:

- **Three-pass approach**: place tooltips → read real heights → stagger → draw arrows → create container
- **Arrow routing**: `buildArrow(pts)` via `arrowH` / `arrowV`
- **Reparenting**: correct local coords after `container.appendChild`
- **Container name**: `🔍 A11y - [Frame name]`
- **Preserve prior runs**: never delete existing A11y annotation containers automatically. Each run creates a new frame-specific container. Let users delete old containers themselves unless they explicitly ask you to remove one.

Use Appendix A selectively in this order:
1. `Tooltip component`
2. `Determining element coordinates`
3. `Complete drawing loop`
4. `Known gotchas` (only if the result is incorrect)

### Side assignment

Sort all annotations by `elementCenterY` (top to bottom). Assign sides by alternating: odd indices (0, 2, 4…) → `right`, even indices (1, 3, 5…) → `left`. Override to `left` unconditionally when the element's horizontal centre is in the left half of the frame (`elementCenterX < frameCenterX`). Fall back to `top` for topmost elements when both columns are full, or `bottom` for bottom-nav elements only. Maximum 5 tooltips per side.

The alternating pattern distributes arrow lines across both sides, preventing arrows from crossing and preventing one side from becoming too dense.

### Tooltip content reference

**No em dashes anywhere.** Use a period to separate two clauses, or rephrase as a single sentence. The em dash character is not allowed in chat messages, proposed plans, summaries, tooltip content, or tooltip titles.

| Annotation type | Tooltip title | Content example |
|-----------------|---------------|-----------------|
| Alt text | `"Promo illustration"` | `"Output: Illustrated trophy cup with confetti"` |
| Combined output | `"Passenger selector"` | `"Output: Set passengers. Current value: 3 passengers"` |
| Role only | `"Add passengers button"` | `"Role: Button"` |
| Role choice | `"Terms row"` | `"Role: Link"` |
| State | `"Details panel"` | `"State: Expanded"` |
| Toggle | `"Notifications switch"` | `"Role: Switch\nState: On"` |
| Live region | `"Results count"` | `"Requirement: Announce when value updates without navigation"` |
| Focus order | `"Promo banner"` | `"Requirement: Reached before the list below"` |
| Optional field | `"Return date field"` | `"Requirement: Can be skipped by assistive technology"` |
| Input label | `"From field"` | `"Role: Text input\nRequirement: Label 'From' must be programmatically associated"` |
| Submit button | `"Search button"` | `"Role: Submit button"` |
| Custom component | `"Price range control"` | `"Role: Slider\nOutput: Price range. Current value: $200 to $500"` |
| Heading | `"Recent searches heading"` | `"Role: Heading\nRequirement: Mark as h2"` |
| Selected tab | `"Stays tab"` | `"Output: Stays, new\nRole: Tab\nState: Selected"` |
| Tablist | `"Travel mode tabs"` | `"Role: Tablist\nRequirement: Each option must expose Role: Tab"` |
| Form field value | `"Travellers field"` | `"Output: Travellers. Current value: 1 traveller"` |
| Form field catch-all | `"Search form fields"` | `"Requirement: Labels must be programmatically associated. Applies to all search form fields."` |

**Do not annotate redundant output.** If a button visibly says `Add passengers` and the component spec already guarantees a button role, skip it. If the role is uncertain, annotate only `Role: Button`. Avoid `Output: Add passengers, click to update, button`.

---

## Stage 8 - Designer actions summary

After drawing annotations, deliver this short summary:

```
## A11y annotations placed - [Frame name]

**[count] tooltips added** (see Figma).

### Designer actions:
- [Up to 5 items that require design work but did not get a tooltip]
- [Any interactive element measured below the 24×24 CSS px touch target minimum (WCAG 2.2 AA), by name]
- If none: `No extra designer actions.`
```

Be specific. Only include items that actually require a decision or action from the designer. Do not restate output, role, or state fields already covered by tooltips. Do not repeat the full annotation plan.

---

## Stage 9 - Review

Take a screenshot of the annotation container and show it to the user:

```
get_screenshot(fileKey, containerNodeId)
```

Ask: **"Do these annotations look right? Tell me what to adjust."**

If the user requests changes:
- **Remove one**: delete the group from the container, re-screenshot.
- **Edit text**: update the annotation's populated fields, rebuild `Content#4002:1`, and use `use_figma` to update the instance.
- **Add one**: insert using the three-pass pattern, append to the existing container.
- **Full redo**: create a new replacement container from Stage 6. Do not delete the old container unless the user explicitly asks.

---

## What this skill does NOT annotate

- Colour contrast - separate audit concern
- Animation timing - out of scope for static mockup review
- Component internals - keyboard patterns, gesture handling, focus trapping belong in component specs
- Anything the user explicitly says to skip

---

# Appendix A - Annotation Drawing Reference (implementation details)

> **Purpose:** Complete reference for placing annotation tooltips beside a Figma feature frame and connecting them to their target elements with routed arrows.
>
> **What belongs here:** tooltip component spec, stagger algorithm, arrow routing function, complete ready-to-run code.
>
> **What does NOT belong here:** accessibility criteria logic, which elements to annotate, or how to interpret design content - those belong in the skill stages above.

---

## Tooltip component

The annotation tooltip is `Specs-TootlipBody` from the Specs components library.

| Property | Value |
|----------|-------|
| Component name | `Specs-TootlipBody` |
| Component key | `5d9da1afbffd7efd927d6b59c9d969b34286594b` |
| Source file key | `rPl6NXhJ9nU0b7xXfr4jBI` |
| Source node ID | `4002:2156` |
| Default size | 279 × 50 px |
| Background colour | `#d12771` - `{ r: 0.8196, g: 0.1529, b: 0.4431 }` |

### Internal structure (for reference)

| Node name | Role | Default text | Font |
|-----------|------|--------------|------|
| `Tooltip label` | Target element name (small caption above) | `"Ex: Passenger selector"` | GT Eesti Pro Display Medium, 14px |
| `Tooltip content` | Populated accessibility fields | `'Ex: "Output: Set passengers. Current value: 3 passengers"'` | GT Eesti Pro Display Regular, 16px |

### Importing the component and setting text

The component exposes three properties - use `setProperties()` rather than finding raw text nodes. No manual font loading is needed.

```javascript
const component = await figma.importComponentByKeyAsync(
  '5d9da1afbffd7efd927d6b59c9d969b34286594b'
);
const tip = component.createInstance();
const content = buildTooltipContent({
  output: 'Set passengers. Current value: 3 passengers',
  role: 'Button'
});
tip.setProperties({
  'Title#4002:0':   'Passenger selector',
  'Content#4002:1': content,
  'show_title#4002:2': true
});
```

| Property key | Type | Role |
|---|---|---|
| `Title#4002:0` | TEXT | Target element name (small caption) |
| `Content#4002:1` | TEXT | Populated accessibility fields (`Output`, `Role`, `State`, `Requirement`) |
| `show_title#4002:2` | BOOLEAN | Show/hide the title row |

Compose `Content#4002:1` from only the populated fields. Do not render empty labels.

```javascript
function buildTooltipContent(ann) {
  return [
    ['Output', ann.output],
    ['Role', ann.role],
    ['State', ann.state],
    ['Requirement', ann.requirement]
  ]
    .filter(([, value]) => Boolean(value))
    .map(([label, value]) => `${label}: ${value}`)
    .join('\n');
}
```

---

## Layer structure

All annotations for a single run live inside a frame-specific container. Multiple runs must coexist, especially when they concern different screens. Users can delete any container themselves in one click.

```
Frame "🔍 A11y - [Frame name]"  clipsContent=false, no fill, no stroke
  Group "↳ Status bar"          auto-sized to bounding box of its children
    InstanceNode (Specs-TootlipBody)
    VectorNode (arrow)
  Group "↳ Navigation heading"
    InstanceNode
    VectorNode
  …
```

`clipsContent = false` is required because arrow vectors extend left out of the container and into the feature frame.

---

## Supported sides

Tooltips can be placed on any of the four sides of a feature frame. Assign each annotation a `side` before calling the routing or stagger functions.

| Side | Tooltip column/row | Stagger axis | Arrow direction |
|------|--------------------|--------------|-----------------|
| `right` | vertical column to the right | vertical (y) | exits tooltip **left** → enters element **right** |
| `left`  | vertical column to the left  | vertical (y) | exits tooltip **right** → enters element **left** |
| `top`   | horizontal row above         | horizontal (x) | exits tooltip **bottom** → enters element **top** |
| `bottom`| horizontal row below         | horizontal (x) | exits tooltip **top** → enters element **bottom** |

---

## Layout constants

```javascript
const TOOLTIP_W  = 279;   // tooltip width (matches component default)
const TOOLTIP_H  = 50;    // tooltip height (component default / collapsed).
                            // CAUTION: setProperties can grow the tooltip to ~70 px when content
                            // is long. Do NOT use this constant for stagger spacing - always read
                            // actual heights after placement (see "Complete drawing loop").
const MIN_GAP    = 16;    // minimum gap between adjacent tooltip bodies (px) - always enforced
const STRAIGHT_THR = 5;  // treat as straight arrow if displacement ≤ this (px)
```

### Per-side position constants

Replace hardcoded values with frame-relative calculations:

```javascript
// RIGHT side
const RIGHT_TOOLTIP_X   = frameRightEdge  + 64;  // tooltip left edge
const RIGHT_GUTTER_X    = frameRightEdge  + 20;  // vertical elbow channel

// LEFT side
const LEFT_TOOLTIP_RX   = frameLeftEdge   - 40;  // tooltip RIGHT edge (canvas)
const LEFT_GUTTER_X     = frameLeftEdge   - 8;   // vertical elbow channel

// TOP side
const TOP_TOOLTIP_BY    = frameTopEdge    - 40;  // tooltip BOTTOM edge (canvas)
const TOP_GUTTER_Y      = frameTopEdge    - 8;   // horizontal elbow channel

// BOTTOM side
const BOTTOM_TOOLTIP_TY = frameBottomEdge + 40;  // tooltip TOP edge (canvas)
const BOTTOM_GUTTER_Y   = frameBottomEdge + 8;   // horizontal elbow channel
```

The gutter sits in the gap between the tooltip and the frame edge, keeping elbowed arrows from crossing UI content.

---

## Stagger algorithm

> **Important:** `setProperties` can grow a tooltip taller than the default 50 px. Never use a fixed height constant for stagger spacing. Always read **actual** heights after placing tooltip instances (see "Complete drawing loop"). The functions below show the algorithm logic; the drawing loop is the authoritative implementation.

`MIN_GAP` is a hard floor: every adjacent pair of tooltip bodies is separated by at least `MIN_GAP` pixels, regardless of the natural spread of the annotated elements.

### Vertical stagger (right / left sides)

Sort annotations by `elementCenterY` ascending, cascade downward - using **actual heights** `h` read from `tip.height` after `setProperties`:

```javascript
// items is already sorted by elementCenterY
let prevBottom = -Infinity;
for (const { ann, tip } of items) {
  const h = tip.height;                           // actual height - may be > 50
  const desired = ann.elementCenterY - h / 2;
  const actual  = Math.max(desired, prevBottom + MIN_GAP);
  tip.y = actual;
  prevBottom = actual + h;                        // track true bottom
}
```

### Horizontal stagger (top / bottom sides)

Same algorithm, axis swapped - sort by `elementCenterX`, stack rightward. Read `tip.width` after `setProperties` (width is fixed for this component so the constant is reliable, but reading it is still cleaner):

```javascript
function staggerHorizontal(annotations) {
  const sorted = [...annotations].sort((a, b) => a.elementCenterX - b.elementCenterX);
  let prevRight = -Infinity;
  return sorted.map(ann => {
    const desiredX = ann.elementCenterX - TOOLTIP_W / 2;
    const actualX  = Math.max(desiredX, prevRight + MIN_GAP);
    prevRight = actualX + TOOLTIP_W;
    return { ...ann, tipX: actualX, tipCenterX: actualX + TOOLTIP_W / 2 };
  });
}
```

Any tooltip pushed from its natural position automatically gets an elbowed arrow - no extra logic.

---

## Determining element coordinates

`elementEdgeX/Y` and `elementCenterX/Y` must be the **target element's own bounding box**, not its container's or the frame's edge.

### For top-level frame children

Read directly from `get_metadata` results:
```javascript
// get_metadata returns: <instance x="24" y="12" width="327" height="120" />
// Canvas coords = frame origin + local offset
const elementEdgeX  = FRAME_X + x + width;   // right edge
const elementCenterY = FRAME_Y + y + height / 2;
```

### For sub-elements inside component instances

`get_metadata` does not expand component instances - it returns only the instance's outer bounding box. To get sub-element positions, derive them from the design context padding analysis:

1. Read `get_design_context` output for the instance.
2. Trace the CSS padding chain (e.g. `px-24` on the outer wrapper, `px-16` on the inner column).
3. Subtract total padding from the frame edge to find the sub-element's right edge.

Example - a form inside a full-width component with outer padding 24px and inner padding 16px:
```javascript
const formElementEdgeX = FRAME_RIGHT - 24 - 16;  // = FRAME_RIGHT - 40
```

### Common mistake

Do not default to `FRAME_RIGHT` (or `FRAME_LEFT/TOP/BOTTOM`) for all annotations. An element that is visually inset from the frame edge will produce an arrow that floats in empty space beside the screen, pointing at nothing.

---

> **Critical:** Figma normalises vector vertex coordinates to be non-negative by shifting them to the bounding-box top-left. If you pass any vertex with a negative `x` or `y`, Figma moves it to `(0, 0)` and shifts the other vertices accordingly - **without** adjusting `vec.x / vec.y`. The arrow then lands at the wrong canvas position.
>
> **Fix:** always compute all path points in canvas coordinates first, find the bounding-box minimum, place the vector there, and express all vertex coords as non-negative offsets from that minimum. The functions below do this automatically.

### Canonical helper: `buildArrow`

```javascript
// pts: array of canvas-absolute {x, y} points. Last point gets the arrowhead.
// Returns { network, vecX, vecY } - place vec at (vecX, vecY).
function buildArrow(pts) {
  const minX = Math.min(...pts.map(p => p.x));
  const minY = Math.min(...pts.map(p => p.y));
  const n = pts.length;
  return {
    network: {
      vertices: pts.map((p, i) => ({
        x: p.x - minX,
        y: p.y - minY,
        strokeCap: i === n - 1 ? 'ARROW_LINES' : 'NONE'
      })),
      segments: pts.slice(0, -1).map((_, i) => ({ start: i, end: i + 1 })),
      regions: []
    },
    vecX: minX,
    vecY: minY
  };
}
```

### Horizontal sides (right / left)

The arrowhead is always on the **element** end (last point).

```javascript
// tipEdgeX: canvas x of the tooltip edge facing the element
//   RIGHT → tip.x           (left edge of tooltip)
//   LEFT  → tip.x + tip.width  (right edge of tooltip)
// tipCY: tip.y + tip.height / 2  (vertical centre of tooltip)
function arrowH(tipEdgeX, tipCY, elEdgeX, elCY, gutterX) {
  const pts = Math.abs(tipCY - elCY) <= STRAIGHT_THR
    ? [ { x: tipEdgeX, y: elCY   },
        { x: elEdgeX,  y: elCY   } ]
    : [ { x: tipEdgeX, y: tipCY  },
        { x: gutterX,  y: tipCY  },
        { x: gutterX,  y: elCY   },
        { x: elEdgeX,  y: elCY   } ];
  return buildArrow(pts);
}
```

### Vertical sides (top / bottom)

```javascript
// tipCX: tip.x + tip.width / 2  (horizontal centre of tooltip)
// tipEdgeY: canvas y of the tooltip edge facing the element
//   TOP    → tip.y + tip.height  (bottom edge of tooltip)
//   BOTTOM → tip.y               (top edge of tooltip)
function arrowV(tipCX, tipEdgeY, elCX, elEdgeY, gutterY) {
  const pts = Math.abs(tipCX - elCX) <= STRAIGHT_THR
    ? [ { x: elCX,   y: tipEdgeY },
        { x: elCX,   y: elEdgeY  } ]
    : [ { x: tipCX,  y: tipEdgeY },
        { x: tipCX,  y: gutterY  },
        { x: elCX,   y: gutterY  },
        { x: elCX,   y: elEdgeY  } ];
  return buildArrow(pts);
}
```

Arrow stroke colour **must match** the tooltip background:

```javascript
const tipColor = { r: 0.8196, g: 0.1529, b: 0.4431 }; // hardcoded from component
```

---

## Complete drawing loop

**Use a three-pass approach: place all tooltip bodies → re-stagger with actual heights → draw arrows.**

Do not compute arrow coordinates from the stagger output alone - `setProperties` can resize tooltips, and the stagger uses `TOOLTIP_H = 50` as an initial estimate. After placing all tooltips, read their **actual** heights and re-run the stagger before drawing arrows. This guarantees correct spacing and accurate arrow origins regardless of how the component laid out.

```
Pass 1a: place all tooltip instances (rough initial y)
Pass 1b: read tip.height for each, re-run stagger with actual heights, set tip.y
Pass 2:  draw arrows from final tip.x / tip.y / tip.height
Pass 3:  create container, reparent, group
```

```javascript
async function drawAnnotations(annotations, page, frameName) {
  // annotations: [{label, output?, role?, state?, requirement?, side, elementEdgeX/Y, elementCenterX/Y}]

  // Preserve previous annotation containers. Each run creates a new frame-specific container.
  // Only delete an existing container when the user explicitly asks.

  const component = await figma.importComponentByKeyAsync(COMP_KEY);

  // ── PASS 1: place all tooltip bodies on the page ──────────────────────
  // Stagger per side to avoid overlaps
  const byRight  = staggerVertical(annotations.filter(a => a.side === 'right'));
  const byLeft   = staggerVertical(annotations.filter(a => a.side === 'left'));
  const byTop    = staggerHorizontal(annotations.filter(a => a.side === 'top'));
  const byBottom = staggerHorizontal(annotations.filter(a => a.side === 'bottom'));

  const items = []; // { ann, tip }

  for (const ann of byRight) {
    const tip = component.createInstance();
    tip.x = frameRightEdge + 64;         // tooltip left edge
    tip.y = ann.tipY;
    page.appendChild(tip);
    tip.setProperties({'Title#4002:0':ann.label,'Content#4002:1':buildTooltipContent(ann),'show_title#4002:2':true});
    items.push({ ann, tip });
  }
  for (const ann of byLeft) {
    const tip = component.createInstance();
    tip.x = (frameLeftEdge - 40) - TOOLTIP_W;  // tooltip left edge; right edge = frameLeft - 40
    tip.y = ann.tipY;
    page.appendChild(tip);
    tip.setProperties({'Title#4002:0':ann.label,'Content#4002:1':buildTooltipContent(ann),'show_title#4002:2':true});
    items.push({ ann, tip });
  }
  for (const ann of byTop) {
    const tip = component.createInstance();
    tip.x = ann.tipX;
    tip.y = (frameTopEdge - 40) - TOOLTIP_H;   // tooltip top edge; bottom edge = frameTop - 40
    page.appendChild(tip);
    tip.setProperties({'Title#4002:0':ann.label,'Content#4002:1':buildTooltipContent(ann),'show_title#4002:2':true});
    items.push({ ann, tip });
  }
  for (const ann of byBottom) {
    const tip = component.createInstance();
    tip.x = ann.tipX;
    tip.y = frameBottomEdge + 40;              // tooltip top edge
    page.appendChild(tip);
    tip.setProperties({'Title#4002:0':ann.label,'Content#4002:1':buildTooltipContent(ann),'show_title#4002:2':true});
    items.push({ ann, tip });
  }

  // ── PASS 2: read REAL positions, draw arrows ──────────────────────────
  // arrowH / arrowV return { network, vecX, vecY } - vecX/vecY is the
  // bounding-box min of the path, ensuring all vertex coords are ≥ 0.
  const pairs = [];

  for (const { ann, tip } of items) {
    const tx = tip.x, ty = tip.y;   // page-level node → x/y ARE canvas coords
    const tw = tip.width, th = tip.height;

    let arrow;
    if (ann.side === 'right')
      arrow = arrowH(tx,    ty+th/2, ann.elementEdgeX,  ann.elementCenterY, frameRightEdge+20);
    else if (ann.side === 'left')
      arrow = arrowH(tx+tw, ty+th/2, ann.elementEdgeX,  ann.elementCenterY, frameLeftEdge-8);
    else if (ann.side === 'top')
      arrow = arrowV(tx+tw/2, ty+th, ann.elementCenterX, ann.elementEdgeY,  frameTopEdge-8);
    else
      arrow = arrowV(tx+tw/2, ty,    ann.elementCenterX, ann.elementEdgeY,  frameBottomEdge+8);

    const vec = figma.createVector();
    vec.name = `↳ ${ann.label}`;
    await vec.setVectorNetworkAsync(arrow.network);
    vec.x = arrow.vecX;  vec.y = arrow.vecY;   // place at bounding-box min
    vec.strokes = [{ type: 'SOLID', color: TIP_COLOR }];
    vec.strokeWeight = 2.5;
    vec.fills = [];
    page.appendChild(vec);
    pairs.push({ tip, vec, tX: tx, tY: ty, vX: arrow.vecX, vY: arrow.vecY });
  }

  // ── PASS 3: create container, reparent, group ─────────────────────────
  // ⚠️  `appendChild` does NOT preserve canvas position.
  // After reparenting, node.x/y are kept unchanged but are now interpreted as
  // local coords relative to the container. Fix: subtract container origin.
  const allNodes = pairs.flatMap(({ tip, vec }) => [tip, vec]);
  const boxes    = allNodes.map(n => n.absoluteBoundingBox);
  const FX = Math.min(...boxes.map(b => b.x)) - 8;
  const FY = Math.min(...boxes.map(b => b.y)) - 8;
  const FR = Math.max(...boxes.map(b => b.x + b.width)) + 8;
  const FB = Math.max(...boxes.map(b => b.y + b.height)) + 8;

  const container = figma.createFrame();
  container.name         = `🔍 A11y - ${frameName}`;
  container.x = FX; container.y = FY;
  container.fills        = [];
  container.strokes      = [];
  container.clipsContent = false;
  container.resize(FR - FX, FB - FY);
  page.appendChild(container);

  for (const { tip, vec, tX, tY, vX, vY } of pairs) {
    // Save canvas coords before reparenting, then correct to container-local
    container.appendChild(tip); tip.x = tX - FX; tip.y = tY - FY;
    container.appendChild(vec); vec.x = vX - FX; vec.y = vY - FY;
    figma.group([tip, vec], container);
  }

  return container;
}
```

---

## Known gotchas

| Situation | Fix |
|-----------|-----|
| `figma.createConnector()` not available | Design file plugins don't expose connectors; use `createVector` + `setVectorNetworkAsync` instead |
| Arrow appears as double-headed | `strokeCap` on a `LineNode` applies to both ends; use `VectorNode` with per-vertex `strokeCap` |
| Arrowhead points away from element | Ensure the element end is the **last vertex** (`ARROW_LINES`) and `vec.x/y` is placed at `elementRightX, elementCenterY` |
| Arrows disappear inside container | Set `container.clipsContent = false` - arrows extend left beyond the container bounds |
| `figma.group()` silently fails | Both nodes must already be children of the target parent before calling `figma.group([a, b], parent)` |
| `GUTTER_X` cuts through frame content | Set dynamically: right `frameRightEdge + 20`, left `frameLeftEdge − 8`, top `frameTopEdge − 8`, bottom `frameBottomEdge + 8` |
| Component not found on import | Ensure the Specs components library (`rPl6NXhJ9nU0b7xXfr4jBI`) is enabled for the target file's team |
| Elbowed arrow disconnected from tooltip | The critical vertex bug: elbow vertex 0 must be `{ x: rx, y: dy }` (at tooltip y) and vertex 3 must be `{ x: 0, y: 0 }` (at element y, arrowhead). Using `y: 0` at vertex 0 and `y: -dy` at vertex 3 inverts the path, making it start at element level and point away from both tooltip and element. |
| Top / bottom arrows cross UI content | Top and bottom arrows travel vertically through the mockup to reach their target's top/bottom edge. This is inherent to vertical routing. Prefer right/left sides when possible; use top/bottom only when the right and left columns are full, or for elements at the very top or bottom of a tall frame. |
| Arrow centre slightly above tooltip visual centre | `tipCenterY` in the stagger uses `TOOLTIP_H = 50`, but `setProperties` can grow the tooltip to ~70 px. The arrow will start ~10 px above the visual centre but still within the tooltip body - acceptable unless pixel-perfect alignment is required. Fix by reading `tip.height` after a short `await` post-`setProperties` and re-running `routeArrow`. |
| Left / top arrows land in completely wrong position | **Figma vertex normalisation bug.** `setVectorNetworkAsync` silently shifts all vertex coordinates to be ≥ 0 (origin = bounding-box min) - but does **not** adjust `vec.x / vec.y`. Any vertex with a negative `x` or `y` (which occurs naturally for left-side and top-side arrows when the vector origin is at the element edge) will be mis-placed. **Fix:** use `buildArrow(pts)` - compute all path points in canvas-absolute coords, derive `vecX/vecY` as the bounding-box minimum, then express all vertex coords as `(pt.x − vecX, pt.y − vecY)`. All coordinates are then guaranteed ≥ 0. |
| Annotations jump to wrong position after reparenting into container | **`appendChild` does not preserve canvas position.** When a page-level node at canvas `(x, y)` is reparented to a container at `(FX, FY)`, the node's `x`/`y` values are kept unchanged but are now interpreted as container-local coords, so canvas position becomes `(FX + x, FY + y)`. **Fix:** record canvas coords before `appendChild`, then correct after: `container.appendChild(node); node.x = canvasX − FX; node.y = canvasY − FY`. |