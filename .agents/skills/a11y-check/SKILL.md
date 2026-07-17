---
name: a11y-check
description: Produce minimal, actionable accessibility annotations for a single feature screen in Figma. Places tooltip annotations beside the mockup and delivers a written summary of designer actions. Use when asked to annotate, audit, or review a Figma screen for accessibility.
compatibility: Requires Figma MCP (mcp_figma_* tools)
---

# A11y Check — Skill

> **Purpose:** Produce a minimal, actionable set of accessibility annotations for a single feature screen. Place tooltip annotations beside the Figma mockup and deliver a written summary of designer actions not covered by tooltips.
>
> **What this skill does NOT do:** audit the full design system. It focuses on the *feature layer* — the specific arrangement, content, and context of elements on this screen.

---

## Design principle: minimal annotations

The goal is the **fewest annotations that carry real value**. Every tooltip is a decision the designer must act on. Annotation fatigue defeats the purpose.

Only annotate what is ambiguous, missing, or bespoke to this screen. Skip anything handled natively by assistive technology or already fully covered by a component spec.

## Platform scope

Annotate for **web (WCAG AA), native iOS, and native Android** in every run. Annotations describe the accessibility **requirement** — not the implementation. Do not include platform-specific syntax, APIs, or code in tooltip content. Leave implementation to developers.

- ✅ `"Value must be announced with its unit"` — platform-agnostic
- ❌ `"Use aria-label='87 points'"` — web-only implementation

---

## Scope: one frame per run

Run this skill once per Figma frame. For multi-screen flows, run once per screen — containers are named after the frame and coexist cleanly.

---

## Stage 1 — Acquire the frame

Require a Figma URL with a `node-id` before proceeding:

```
https://www.figma.com/design/{fileKey}/{fileName}?node-id={nodeId}
```

If the URL points to a page root (no `node-id`), ask the user to select a specific frame. If the user provides multiple screens, ask them to pick one.

---

## Stage 1b — Verify MCP availability

Run both probes **in parallel** immediately after extracting `fileKey` and `nodeId`. Use the lightest call available for each MCP.

| Probe | Call | Reused in |
|-------|------|-----------|
| Figma | `get_metadata(fileKey, nodeId)` | Stage 2 — do **not** call again |
| Supernova | `get_documentation_page_list()` | Stage 4 — do **not** call again |

| Result | Action |
|--------|--------|
| Figma unavailable (tool not found, auth error, timeout) | **Hard stop.** Tell the user the Figma MCP is not reachable and do not proceed. |
| Supernova unavailable | **Soft fail.** Warn the user once, then continue — activate the fallback path in Stage 4. |
| Both succeed | Cache both results and continue. |

**Figma hard-stop message:**
> "I can't reach the Figma MCP (error: `[error text]`). Check that the MCP server is connected and authenticated — see the Setup section of the README — then start a new conversation."

**Supernova soft-fail message:**
> "Supernova is not reachable right now. Component coverage checks will be skipped — I'll ask you directly about any spec gaps as we go."

---

## Stage 2 — Gather context

`get_metadata` is already available from Stage 1b — **do not call it again**. Only call `get_design_context`:

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

## Stage 3 — Analyse for WCAG AA requirements

Walk the layer tree and ask: **"What would need to be done to make this screen accessible?"** across web (WCAG AA), native iOS, and native Android.

For each element, identify concerns — do not filter yet. Common areas to check:

- Interactive elements: is the role clear? Is the name derivable?
- Form fields: are labels programmatically associated?
- Imagery: meaningful or decorative?
- State: communicated beyond colour alone?
- Touch targets: meet 44×44 pt minimum?
- Focus/reading order: logical for the layout?
- Live regions: anything that updates without navigation?
- Component roles: custom controls that need explicit role declaration?

Produce an internal list of **concerns**, each with: element, concern type, and a preliminary requirement statement.

### Hard skips (do not raise as concerns)

| Condition | Reason |
|-----------|--------|
| Plain body text, labels, captions (`TEXT` node with no heading role) | Read natively by screen readers — no annotation needed |
| Decorative shape / background fill with no image | No semantic content |
| Layout container (`FRAME`/`GROUP`) with no interactive role and no image fill | Pure layout |
| Illustration or Icon confirmed as decorative | Decorative is the default; nothing actionable |
| Component whose entire visible output is readable text (body copy, labels) | Text is announced natively — check via `get_design_context` before raising |
| Anything the user has explicitly said to skip | User has waived it |

> **Do NOT skip section headings.** A text element that visually functions as a section heading (larger or bolder than surrounding text, or whose layer name contains "Heading", "Title", or "H1"–"H6") requires explicit heading role markup — this is not automatic. Annotate it with the `"Heading"` annotation type.

### Batching repeated concerns

When the same requirement applies to **multiple instances of the same component** on the screen, use a single **catch-all annotation** on one representative instance rather than annotating each row individually.

**When to batch:**
- 3 or more instances of the same component share an identical concern
- The requirement is uniform (same label strategy, same role, same pattern) — no instance-specific decision is needed

**How to annotate:**
- Pick the first or most visually prominent instance as the target
- Word the description to make the scope explicit: *"applies to all [Component] rows on this screen"* or *"applies to all Items with supplementary text in this list"*
- Add any instance that has a **unique, screen-specific decision** (e.g. an unusual label combination, an inline action) as a **separate annotation** alongside the catch-all

**Example:** A settings screen with 8 Item/Navigation rows, all needing unified accessible labels → one catch-all annotation on the first row, plus a second annotation on the "Home address" row that also has an inline "Verify" action.

---

## Stage 4 — Component coverage check

For each concern that involves a named component instance:

1. **Batch lookup**: collect all component names, then call `get_design_system_component_list`. Use the `get_documentation_page_list` result already cached from Stage 1b — do **not** call it again.
2. For each match, call `get_documentation_page_content` and **read the Accessibility section in full**.
3. Judge coverage against the specific concern:

| Coverage result | Action |
|----------------|--------|
| Spec fully addresses this concern for this feature usage, **and** the correct answer is unambiguous in this context | **Silent skip** — remove from concern list, no annotation, no user question |
| Spec covers the requirement but the **correct value depends on context** (e.g. role varies by usage, label is screen-specific) | **Keep** — surface as a Stage 5 question |
| Spec partially addresses it or leaves a feature-specific decision open | **Keep** — annotate the gap |
| No spec, spec is empty, or spec is too generic | **Keep** — treat as `needs-decision` |

Do not assume a spec exists or is sufficient. Read it. A spec that says "follow platform accessibility guidelines" without specifics is too generic.

**Context-dependent role example:** A spec that says *"Item/Navigation must have `role="link"` or `role="button"`"* covers the requirement — but which role is correct on *this screen* depends on whether the item navigates to another page (link) or triggers an in-page action (button). That choice is screen-specific and must be confirmed with the user.

**No Supernova redirect tooltips.** Annotations must be self-contained and immediately actionable.

### When Supernova is unavailable

Supernova unavailability was detected and reported in Stage 1b — do not repeat the warning. For each concern that would normally trigger a component lookup, ask the user directly:

> "For **[Component Name]** — does it have an accessibility spec that covers [specific concern]? If yes, tell me what it says. If no, tell me what the requirement should be."

---

## Stage 5 — Clarifying questions

After stages 3–4, ask the user about anything that is still ambiguous. Ask **one question at a time** — wait for the answer before asking the next.

**Question order:**
1. Ambiguous imagery / illustrations (decorative vs meaningful)
2. Standalone icons (redundant vs meaningful)
3. Interactive groups (single focusable unit vs individually traversable)
4. Custom components whose role is unclear
5. **Context-dependent roles** — for any component where the spec lists multiple valid roles (e.g. button vs link), ask which applies on this screen. Offer a catch-all if all instances behave the same way.
6. Focus order (one question for the whole screen, ask last)

### Question format

> **[Layer name]** — [what you observe and why it's ambiguous]. My suggestion: [concrete recommendation]. Is that right?

Use the layer name from `get_metadata`. Keep suggestions concrete so the user can just say "yes" or give a one-line correction.

Accept "out of scope", "predates this feature", or "covered elsewhere" without asking for justification — skip the element and move on.

---

## Stage 6 — Plan

After all questions are resolved, present the proposed annotation plan **before drawing anything**.

Format:

```
## Proposed annotations — [Frame name]

| # | Element | Annotation | Why |
|---|---------|-----------|-----|
| 1 | SupplySelector | Component role | Custom control. Role not declared in spec. |
| 2 | From / To inputs | Input | Labels must be programmatically associated |
| 3 | Return date | Optional field | Skippable, not flagged in spec |
| 4 | Search button | Submit | Primary form action |

**Not annotated (and why):**
- Banner: decorative image — no alt text required
- Illustration (empty state): decorative — default behaviour
- MenuBar: out of scope (your request)

**Designer actions not covered by tooltips:**
- [list anything that requires design decisions but can't fit a tooltip]
```

Then ask: **"Does this plan look right? Anything to add, remove, or change before I draw the annotations?"**

Do not proceed to Stage 7 until the user confirms the plan.

---

## Stage 7 — Place annotations

For each confirmed annotation, finalise:

| Field | Content |
|-------|---------|
| `label` | Short category label (e.g. `"Alt text"`, `"Input"`, `"Submit"`, `"Component role"`) |
| `description` | Specific, platform-agnostic requirement for this element on this screen |
| `side` | Tooltip side (`right`, `left`, `top`, `bottom`) |
| `elementEdgeX/Y` | Canvas coordinate of the **target element's own** edge facing the tooltip — not its container's edge, not the frame edge. Read from `get_metadata` for top-level nodes; derive from padding analysis for sub-elements. See `references/drawing-tooltips.md → Determining element coordinates`. |
| `elementCenterX/Y` | Canvas centre of the element on the axis perpendicular to the arrow |

Then follow `references/drawing-tooltips.md` exactly:

- **Three-pass approach**: place tooltips → read real heights → stagger → draw arrows → create container
- **Arrow routing**: `buildArrow(pts)` via `arrowH` / `arrowV`
- **Reparenting**: correct local coords after `container.appendChild`
- **Container name**: `🔍 A11y — [Frame name]`

### Side assignment

Sort all annotations by `elementCenterY` (top to bottom). Assign sides by alternating: odd indices (0, 2, 4…) → `right`, even indices (1, 3, 5…) → `left`. Override to `left` unconditionally when the element's horizontal centre is in the left half of the frame (`elementCenterX < frameCenterX`). Fall back to `top` for topmost elements when both columns are full, or `bottom` for bottom-nav elements only. Maximum 5 tooltips per side.

The alternating pattern distributes arrow lines across both sides, preventing arrows from crossing and preventing one side from becoming too dense.

### Tooltip content reference

**No em dashes in tooltip text.** Use a period to separate two clauses, or rephrase as a single sentence. Em dashes (`—`) are not allowed in tooltip content or titles.

| Annotation type | Title | Content example |
|-----------------|-------|-----------------|
| Alt text | `"Alt text"` | `"Illustrated trophy cup with confetti"` |
| Combined label | `"Combined label"` | `"Score: 87 out of 100, trending up"` |
| Touch target | `"Touch target"` | `"Below 44×44 pt minimum. Increase tap area."` |
| State | `"State"` | `"Colour alone does not communicate state. Add a text or shape indicator."` |
| Live region | `"Live region"` | `"Announce when value updates without navigation"` |
| Focus order | `"Focus order"` | `"Reached before the list below"` |
| Optional field | `"Optional field"` | `"Can be skipped by assistive technology"` |
| Input label | `"Input"` | `"Label 'From' must be programmatically associated"` |
| Submit button | `"Submit"` | `"Primary form action. Must behave as submit."` |
| Custom component | `"Component role"` | `"Acts as a slider. Announce current value and range."` |
| Heading | `"Heading"` | `"Section title. Must be marked as a heading for screen reader navigation."` |

---

## Stage 8 — Designer actions summary

After drawing annotations, deliver a concise written summary:

```
## A11y annotations placed — [Frame name]

**[count] tooltips added** (see Figma).

### What the tooltips don't cover — designer actions required:
- [Each item that requires design work but didn't get a tooltip, e.g.:]
  - "Confirm touch target sizes meet 44×44 pt for all interactive elements in the header"
  - "Verify the banner image alt text is agreed with content team before handoff"
  - "Check that colour contrast on the Search button meets 4.5:1 (AA) — not checked here"
```

Be specific. Only include items that actually require a decision or action from the designer.

---

## Stage 9 — Review

Take a screenshot of the annotation container and show it to the user:

```
get_screenshot(fileKey, containerNodeId)
```

Ask: **"Do these annotations look right? Anything to adjust, add, or remove?"**

If the user requests changes:
- **Remove one**: delete the group from the container, re-screenshot.
- **Edit text**: use `use_figma` to update `Content#4002:1` on the instance.
- **Add one**: insert using the three-pass pattern, append to the existing container.
- **Full redo**: clear the container and re-run from Stage 6 with updated decisions.

---

## What this skill does NOT annotate

- Colour contrast — separate audit concern
- Animation timing — out of scope for static mockup review
- Component internals — keyboard patterns, gesture handling, focus trapping belong in component specs
- Anything the user explicitly says to skip
