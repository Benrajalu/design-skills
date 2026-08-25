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

## Design principle: minimal annotations

The goal is the **fewest annotations that carry real value**. Every tooltip is a decision the designer must act on. Annotation fatigue defeats the purpose.

Only annotate what is ambiguous, missing, or bespoke to this screen. Skip anything handled natively by assistive technology or already fully covered by a component spec.

See `../../../docs/accessibility-annotation-guide.md` for the shared, designer-facing reference this skill follows: what to annotate vs skip, the annotation field format with examples, and the roles table.

## Conversation style

Keep chat with the designer brief and decision-focused. Do not narrate internal analysis, MCP calls, layer-tree observations, WCAG theory, or platform background unless the user asks or a decision depends on it.

Use one-line progress updates only at stage boundaries or when blocked.

- Verbose: `I am now checking the component documentation to understand whether the accessibility spec covers this concern.`
- Preferred: `Checking component coverage.`

Ask only necessary questions, and keep each question to one short sentence plus a recommendation.

- Verbose: `I noticed this calendar icon might be meaningful because users may need to understand that it opens a date picker. Should it be announced or treated as decorative?`
- Preferred: `Calendar icon: decorative? My suggestion: yes.`

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
| `elementEdgeX/Y` | Canvas coordinate of the **target element's own** edge facing the tooltip - not its container's edge, not the frame edge. Read from `get_metadata` for top-level nodes; derive from padding analysis for sub-elements. See `references/drawing-tooltips.md → Determining element coordinates`. |
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

Then follow `references/drawing-tooltips.md` exactly:

- **Three-pass approach**: place tooltips → read real heights → stagger → draw arrows → create container
- **Arrow routing**: `buildArrow(pts)` via `arrowH` / `arrowV`
- **Reparenting**: correct local coords after `container.appendChild`
- **Container name**: `🔍 A11y - [Frame name]`
- **Preserve prior runs**: never delete existing A11y annotation containers automatically. Each run creates a new frame-specific container. Let users delete old containers themselves unless they explicitly ask you to remove one.

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
