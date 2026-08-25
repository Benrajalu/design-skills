## Documentation component Figma Reference

When documenting accessibility, import the **Accessibility template** component from the remote library using `figma.importComponentByKeyAsync` with its component key.

- **Library**: Specs-components-library v1.2
- **Component key**: `4eaf4591827a8ec8b1127e0ba1ef6c72d47faf19`
- **Node ID** (for reference only): `6193:4389`
- **Link**: [Open component in Figma](https://www.figma.com/design/rPl6NXhJ9nU0b7xXfr4jBI/Specs-components-library---v1.2?node-id=6193-4389)

### How to insert it

The template **replaces** the existing Accessibility section entirely. Do not insert it inside or alongside the old section.

```js
// 1. Import the component from the remote library by its key
const comp = await figma.importComponentByKeyAsync("4eaf4591827a8ec8b1127e0ba1ef6c72d47faf19");

// 2. Find the existing Accessibility section in the parent column and note its index
const column = figma.getNodeById("<component-structure-column-id>");
const oldSection = column.children.find(c => c.name === "Accessibility");
const idx = column.children.indexOf(oldSection);

// 3. Create the instance and insert it AT THE SAME INDEX in the column (not inside the old section)
const instance = comp.createInstance();
column.insertChild(idx, instance);

// 4. IMMEDIATELY detach the instance so its text nodes can be edited directly
instance.detachInstance();
// Note: detachInstance() changes the node ID. Re-query the column's children by name
// ("Accessibility template") to find the new frame — do not reuse the old instance reference.

// 5. Delete the old Accessibility section
oldSection.remove();

// 6. Resize the template to match the column width
const template = column.children.find(c => c.name === "Accessibility template");
template.resize(column.width, template.height);
```

**Critical rules:**
- **Replace, do not nest.** The template goes at the column level, not inside any existing section. Inserting it inside an old section creates broken nested headers and broken layout.
- **Resize immediately after detaching.** The template was designed at 1410px. At narrower column widths (e.g. 950px), resize the outer frame to match the column before filling any text — otherwise text node wrap-widths are locked in at the wrong size.
- **Fill text only after resizing.** Setting `characters` on a text node locks its wrap width at the current rendered width. If you set text before the frame has its final width, the text will wrap at the wrong size and produce enormous node heights. Resize first, fill text second.
- **Do not write to nodes outside the template.** All content — description, guidelines, state data — goes into the template's placeholder nodes only.

After detaching and resizing, traverse the frame's full subtree to collect nodes by name and placeholder content. The current template is state-example-first:

- Summary text lives in the first `Summary + general guidance` block.
- Each relevant state or archetype uses one `#state-template`.
- Each `#state-template` contains one `Example of state with annotations` frame and one `#optional-description`.
- Generic implementation details live in the lower `Technical appendix` block.

### Template structure after detaching

The detached frame contains one reusable `#state-template`. Duplicate it for every meaningful accessibility state or state archetype you need to explain.

Current structure:

```
#state-template
└── (Title + optional description) + Preview
    ├── #state-title
    ├── Example of state with annotations
    └── #optional-description
```

Rules:
- Duplicate `#state-template`, not inner child frames, when adding another state.
- Rename the duplicate title to the state or archetype, e.g. "Empty", "Filled", "Error", "Disabled", "Trailing action".
- Populate `Example of state with annotations` with a linked instance of the exported main component plus contextual annotations.
- Draw those annotations with the shared `Specs-TootlipBody` tooltip and routed vector arrows from `../../../../docs/accessibility-annotation-presentation.md`.
- Put generic or platform-specific details in `#optional-description`, not in the annotation labels.
- Use the lower `Technical appendix` for details that apply across all examples.

### Example of state with annotations — component instances

Every `#state-template` has an `Example of state with annotations` frame. **Always populate it** with a viable exported component instance showing the state being documented.

The top-level example instance must come from the public component in **Main component showcase**. A private primitive can be nested inside that instance, but it must not be the example root.

```js
// 1. Get the exported component or variant from Main component showcase
const mainComp = figma.getNodeById("<main-showcase-component-or-variant-id>");
if (mainComp.type !== "COMPONENT") throw new Error("Example source must be an exported component variant");

// 2. Create an instance and set its variant to match the state
const instance = mainComp.createInstance();
exampleFrame.appendChild(instance);

// For variant-based state (e.g. state=disabled):
instance.setProperties({ state: "disabled" });

// If a state is only available on a nested primitive, keep the exported
// component as the root and override the nested primitive inside it.
const nestedPrimitive = instance.findOne(n => n.type === "INSTANCE" && n.name.includes("/."));
nestedPrimitive?.setProperties({ state: "error" });

// 3. Center the instance in the example frame
instance.x = Math.round((exampleFrame.width - instance.width) / 2);
instance.y = Math.round((exampleFrame.height - instance.height) / 2);
```

**Rules:**
- Always create the example root from an exported main component or variant in Main component showcase; do not draw approximations.
- Do not use primitives from Primitives, Primitives list, or Main Primitive wrapper as top-level examples.
- Do not use private component names such as `Component/.Part` as the example root.
- If the public component does not expose a state directly, create the public component instance first and override the nested primitive inside it.
- Use real component properties and editable text to show the state: placeholder text, selected values, error messages, disabled state, selected/expanded/checked state, loading text, etc.
- Prefer realistic content that makes the accessibility requirement obvious.
- Place the component instance before annotations so annotation shapes stay visible on top.
- For state variants (error, disabled, loading, selected, expanded), use the actual variant/property whenever available.
- If the relevant state requires text but the component instance exposes nested text overrides, edit those text nodes after loading fonts.

### Contextual annotations in examples

Annotations explain the example using the same concise field model as `a11y-check`, but they are generated directly by component-doc.

Presentation must follow `../../../../docs/accessibility-annotation-presentation.md`:
- Import `Specs-TootlipBody` with component key `5d9da1afbffd7efd927d6b59c9d969b34286594b`.
- Set tooltip text through the component properties, not by drawing custom frames or editing arbitrary text boxes.
- Draw connectors as routed vectors with arrowheads on the target element end. Do not use `figma.createLine()` for finished annotations.
- Point every arrow at the exact element described by the populated fields.
- Inside `Example of state with annotations`, set the frame to `layoutMode = "NONE"` and `clipsContent = false`, place the linked component instance first, then place tooltip instances and routed arrows.

Use these fields:
- `Output`: what should be announced when it differs from, combines, or adds to visible text.
- `Role`: semantic role when it is important or ambiguous.
- `State`: exposed state such as selected, expanded, checked, disabled, invalid, loading.
- `Requirement`: behavior, relationship, or decision that is not covered by the other fields.

Formatting rules:
- Use one line per populated field.
- Do not collapse several list-like requirements into a single paragraph.
- Omit empty fields.
- Keep labels short and implementation-agnostic.
- Put platform APIs and detailed implementation notes in `#optional-description` or Technical appendix.

Example annotation text:

```js
Output: Email. Current value: user@example.com
Role: Select field
State: Invalid
Requirement: Error text is announced with the field.
```

Annotation placement rules:
- Point annotations at the exact part of the state example they describe.
- Every visual annotation must use the shared tooltip component. Do not draw custom magenta boxes or manual text callouts.
- Every connector must be a routed vector arrow using the shared helper. Do not use plain `LineNode` connectors.
- If one state has multiple important requirements, add multiple annotations rather than one dense block.
- For single-stop components, annotate the component surface.
- For compound components, annotate each independent focus stop when the difference matters.
- Decorative icons and passive slots should not get annotations unless their treatment is ambiguous.
- If no visual annotation is needed for a generic rule, write it in `#optional-description`.

### Sizing after layout changes

If you need to rearrange layout (e.g. switching `#state-template` from HORIZONTAL to VERTICAL):
- After switching `layoutMode`, set `layoutSizingVertical = "HUG"` on `#state-template` itself and on every child frame recursively. The frame retains its old `FIXED` height unless explicitly changed.
- Text nodes with `textAutoResize = "HEIGHT"` lock their wrap-width at the moment `characters` is set. If the parent was resized after the text was written, re-apply the correct width: set `textAutoResize = "NONE"`, call `resize(correctWidth, node.height)`, then set `textAutoResize = "HEIGHT"` again.

## Role

You are an accessibility expert generating screen reader specifications for VoiceOver (iOS), TalkBack (Android), and ARIA (Web).

## Design principle: layered output for mixed audiences

The audience overlaps with `a11y-check`: designers, PMs, and engineers who need to decide what matters first, then inspect technical depth only when needed.

Use a two-layer output in the Accessibility section:

1. **Designer summary (primary)**: concise and decision-focused. This is the section most people read.
2. **Technical appendix (secondary)**: full platform detail for implementation and QA.

Do not mix these layers. Keep the summary lightweight and the appendix exhaustive.

## Task

**Before starting, read:** `voiceover.md`, `talkback.md`, `aria.md`.

Analyze a UI component from a Figma link, image, or description. Render accessibility documentation directly in Figma using MCP tools.

First produce a concise designer summary, then add the technical appendix with focus order, component anatomy, and platform-specific properties organized by state.

Do NOT output JSON to the user; all data flows directly into Figma template placeholders referenced at the top of this file.

See `../../../../docs/accessibility-annotation-guide.md` for shared concise language and field taxonomy used across skills.

## Inputs

### Figma Link (preferred)
When provided, extract `fileKey` and `nodeId` from the URL, then use MCP tools to gather context:
1. `mcp_figma_get_design_context` — Primary tool: returns component structure, variants, states, reference code, and screenshot
2. `mcp_figma_get_screenshot` — Capture the component visually when a separate screenshot is needed
3. `mcp_figma_get_metadata` — Get structural overview (node IDs, layer types, positions) when full design context is too large
4. `mcp_figma_get_context_for_code_connect` — Get component metadata with properties and variant tree (if nodeId known)
5. `mcp_figma_search_design_system` — Find component by name if URL points to a page rather than a specific node

### Image
Alternative to Figma link. Analyze: element type, visible states, text labels, icons, grouping context.

### Description
User-provided: component type, states to document, context.

### Conflicts

| Scenario | Action |
|----------|--------|
| Description incomplete | Infer from image/Figma; note in `guidelines` |
| Image contradicts description | Description wins |
| Figma link provided | Use MCP tools to supplement visual analysis |

---

## Output contract: summary, annotated states, appendix

### 1) Designer summary (required, primary)

Write this first and keep it short, plain-language, and actionable.

Include only:
- **Component accessibility intent** (1-2 sentences)
- **Focus model summary** (single stop vs multi-stop, traversal gist)
- **State coverage summary** (which states require explicit announcement)
- **Open decisions** (if any unresolved behavior needs product/design confirmation)

Use concise field wording aligned with `a11y-check` where relevant:
- `Output`
- `Role`
- `State`
- `Requirement`

Rules:
- Avoid platform API names in the summary.
- Avoid long property lists.
- Avoid repeating details already visible in component visuals.
- Use line breaks when the summary contains a list of requirements.
- Target readability in under a minute.

### 2) Annotated state examples (required, primary)

This is the most important part of the new template. For every accessibility-relevant state or archetype, duplicate `#state-template` and fill its `Example of state with annotations`.

Include a state example when:
- The state changes what is announced.
- The state changes exposed role or state.
- The state changes focus order or whether an element is focusable.
- The state introduces an error, alert, status, loading message, selected value, expanded content, disabled behavior, or live update.
- The state clarifies a meaningful optional slot, such as a trailing action.

Common examples to consider:
- Empty / placeholder
- Filled / selected value
- Error / invalid
- Disabled
- Focused, only if focus treatment changes the accessibility requirement
- Loading / progress
- Selected / checked / expanded / pressed
- Optional trailing action or close button

Combine examples only when accessibility behavior is identical. Do not combine error and disabled if their announcements or requirements differ.

For each state example:
- Create a linked instance from the exported main component in Main component showcase.
- Set real properties and text overrides to show the state.
- Add annotations using `Output`, `Role`, `State`, and `Requirement`.
- Keep annotation content short and line-broken.
- Add generic implementation notes to `#optional-description`.

### 3) Optional description for each state (secondary)

Use `#optional-description` for details that support the annotated example but should not clutter the visual annotation:
- platform nuance
- merge behavior
- keyboard expectations
- live-region behavior
- known implementation constraints

Use short paragraphs or line breaks. Never write a dense catch-all paragraph when the content is effectively a list.

### 4) Technical appendix (secondary)

Place the full platform-specific content here:
- Top-level focus order model (for compound components)
- VoiceOver / TalkBack / ARIA tables by state
- Detailed property/value/notes rows
- Merge semantics and edge-case handling notes

For trivial single-stop components, keep appendix compact but still include enough implementation detail for handoff.

### 5) Figma structure and emphasis

Within the Accessibility template:
- Place summary content in the first visible block after the title.
- Place annotated state examples before the Technical appendix.
- Keep appendix blocks visually secondary and clearly labeled "Technical appendix".
- Keep appendix in the same frame, but make it obvious it is the deep-dive section.

---

## Analysis Process

### Step 1: List Visual Parts
1. Identify component type (button, checkbox, switch, tab, text field, etc.)
2. **List every visual part** the component contains: label, input, hint text, icon, trailing button, container, divider, etc.

### Step 2: Determine What Gets Merged and What Gets Focus

Most components merge multiple visual parts into a **single focus stop** with one combined announcement. Before determining focus order, analyze which parts merge and which break out as independent stops.

**Ask for each visual part: "Is this an independent focus stop?"**

A part **IS** a focus stop if:
- It's **interactive** — the user can activate, edit, or toggle it (buttons, inputs, links, switches, sliders)
- It's a **container with keyboard navigation** — the container itself is a tab stop with internal arrow-key navigation (tablist, menu, toolbar)

A part is **NOT** a focus stop if:
- It's **merged into another element's announcement** — it provides the accessible name, value, hint, or description for a focusable element (label → input, hint text → input, subtitle → list item)
- It's a **live region** — content appears reactively but the user doesn't navigate to it (error messages, status updates, toast notifications)
- It's **decorative** — dividers, background shapes, non-functional icons

> **Logo-as-text pattern — check before marking any logo as decorative:**
> When a component is part of a family (e.g. branded buttons, payment methods), inspect every variant. Some variants may render the brand name as plain text (logo is decorative), while others render the brand name **only** as a logo image (logo carries the accessible name). These two cases require opposite treatment:
> - **Logo decorative** (brand name present as text): use `accessibilityHidden(true)` / `clearAndSetSemantics {}` / `aria-hidden="true"` on the logo.
> - **Logo is the brand name** (no readable text for the brand): **do NOT hide the logo**. Give it an accessible name (`accessibilityLabel` / `contentDescription` / `alt`) matching the brand name, or set an explicit accessible name on the button that includes the brand name.
> Documenting only one of these cases when both exist is an incomplete spec.

**Merge mechanisms by platform:**

| Platform | How Parts Merge | How Parts Break Out |
|----------|----------------|---------------------|
| iOS | `accessibilityElement = true` on parent; children become part of its label/value/hint | Child with its own `accessibilityElement = true` and interactive trait (`.isButton`) |
| Android | `mergeDescendants = true` (Compose) / `importantForAccessibility = no` on children | Child with `clickable = true` or its own `semantics { }` block |
| Web | Implicit via `<label for>`, `aria-describedby`, `aria-labelledby` | Separate interactive elements (`<button>`, `<a>`, `<input>`) are never merged |

**Common merge patterns:**

| Component | What Merges | Focus Stops |
|-----------|-------------|-------------|
| Text field | Label + input + hint → one stop | Input field; trailing icon button (if interactive) |
| Checkbox + label | Label merges into checkbox | Checkbox only |
| List item (icon + title + subtitle) | All merge into one stop | List item; trailing action button (if present) |
| Chip with close | Label merges into chip body | Chip body; close button |
| Card (heading + description + actions) | Heading + description merge | Card link (if clickable); each action button |
| Tab bar | — | Tablist container; each tab (via arrow keys) |
| Accordion | — | Header/trigger button; content is revealed, not a stop |

**Result:** After this analysis, you have a list of **actual focus stops** — only these go in the `focusOrder` and get their own tables in platform sections. Merged parts are documented as properties (accessible name, hint, value) of the focus stop they're merged into.

### Step 3: Check for Grouping Structure
Ask these diagnostic questions:

1. **Is there a shared label or heading for multiple items?**
   If yes, that label likely names a container that needs a role.

2. **What's the selection model?**
   - Only one can be selected: Group with radio-like semantics
   - Multiple can be selected: Group with checkbox-like semantics
   - Selection switches views/content: Tab-like semantics
   - No selection relationship: Probably not a semantic group

3. **Would "X of Y" positioning be meaningful?**
   If yes, items belong to a countable set; document the container.

4. **Is this a single tab stop with internal arrow navigation?**
   If yes, composite widget; container + children both need documentation.

5. **Would removing the container hurt comprehension?**
   If a screen reader user would be confused hearing items without context, document the group.

**If 2+ questions answer "yes," include the container in the focus order.**

**Do NOT create a container when:**
- Items are visually adjacent but have unrelated purposes
- Each item is independently focusable with no shared selection model
- No platform has a semantic role for this grouping
- The container would just be "Group" with no meaningful label

### Step 4: Enumerate States
List all states to document (enabled, disabled, selected, expanded, error, focused, etc.). For each state, determine if the focus order changes.

### Step 5: Map to Platform Properties
For each focusable part in each state, document the platform-specific properties.

---

## Focus Order Section

For compound components (2+ focusable parts), add a **focus order section** once at the top of the technical appendix. This provides a platform-agnostic overview of the traversal sequence before diving into platform-specific details.

### When to Use

Add a focus order section when the component has **2+ actual focus stops** (as determined by the merge analysis in Step 2). Count only elements a user **lands on** — not parts that are merged into another stop's announcement.

**Include focus order:** Text field with input + trailing icon button (2 stops), tab bar with tablist + tabs (2+ stops), chip with close button (2 stops).

**Omit focus order:** Simple button (1 stop), checkbox with label (1 stop — label merges), toggle switch (1 stop), plain list item without action buttons (1 stop).

> Single-stop components do not need a focus-order example unless the state itself changes focus behavior. Document their accessibility through annotated state examples instead.

### How to Structure

The focus order section uses the same table format as platform sections, but:
- The `title` is `"Focus order"`
- Each table represents one **actual focus stop** in traversal order
- `focusOrderIndex` is the step number (1, 2, 3)
- `name` is the focus stop name (e.g., "Input field", "Trailing icon button")
- `announcement` is a brief description of the stop
- The `properties` describe what visual parts merge into this stop and how

**Important:** Only list actual focus stops. Do not list merged/consumed parts as separate entries. Instead, note them in the `notes` of the stop they merge into.

### Example

Focus order for a text field with trailing icon (2 stops):

- **title**: "Focus order"
- **description**: "Label and hint text merge into the input field's announcement. The trailing icon button is an independent focus stop when present."

| `focusOrderIndex` | `name` | `announcement` | property: type | Notes |
|-------------------|--------|---------------|----------------|-------|
| 1 | Input field | Main interactive element | Focusable | Label and hint text merge into this stop's announcement (not separate focus stops). |
| 2 | Trailing icon button | Independent interactive action | Focusable | E.g., clear button, password toggle. Only present when component includes an interactive trailing action. |

---

## Platform Properties

**Always include role:** `accessibilityTraits` (iOS), `role` (Android), `role` or native element (Web).

**Native form controls:** For text fields, checkboxes, and other native inputs, the role may be implicit. Document the native element (e.g., `<input type="text">`, `UITextField`) and note that role is inherited. Be consistent across all states of the same component.

### iOS (VoiceOver)
Order: Label -> Value -> Traits -> Hint

| Property | Purpose |
|----------|---------|
| `accessibilityLabel` | Spoken name |
| `accessibilityValue` | Current value |
| `accessibilityTraits` | Role/state (`.isButton`, `.isSelected`) |
| `accessibilityHint` | Non-obvious actions only |

### Android (TalkBack)
Order: Content -> Role -> State -> "double-tap to activate"

| Property | Purpose |
|----------|---------|
| `contentDescription` | Spoken label |
| `stateDescription` | State ("checked", "expanded") |
| `role` | Semantic role (`Role.Button`) |

### Web (ARIA)
Order: Name -> Role -> State. Prefer native HTML over ARIA.

| Property | Purpose |
|----------|---------|
| `role` | ARIA role (`"button"`, `"tab"`) |
| `aria-label` | Name when no visible text |
| `aria-selected/expanded/pressed` | State |

---

## Data Structure Reference

*Use this structure to organize your analysis. The data is passed directly into Figma template placeholders — no JSON output is needed.*

```typescript
interface ScreenReaderData {
  componentName: string;
  summary: SummaryData;            // Designer-facing primary layer
  compSetNodeId: string;            // Figma node ID of the component set (from extraction)
  rootSize: { w: number; h: number }; // Default variant dimensions (from extraction)
  elements: FocusElement[];         // All direct children with bounding boxes (from extraction)
  guidelines: string;
  focusOrder?: FocusOrderData;    // Top-level, shown once (compound components only)
  examples: StateExampleData[];    // One rendered #state-template per relevant state/archetype
  technicalAppendix?: string;      // Generic details that apply across examples
}

interface SummaryData {
  intent: string;                  // 1-2 sentences, plain language
  focusModel: string;              // Single-stop vs multi-stop traversal summary
  stateCoverage: string;           // Which states are announced and why
  openDecisions?: string[];        // Only unresolved product/design decisions
}

interface FocusElement {
  index: number;
  name: string;
  bbox: { x: number; y: number; w: number; h: number };
  isFocusStop: boolean;             // true if this element is an actual focus stop (set during merge analysis)
}

interface FocusOrderData {
  title: string;                  // Always "Focus order"
  description?: string;           // Optional description shown under the title (e.g., merge summary)
  stops: FocusOrderStop[];        // Actual focus stops in traversal order
}

interface FocusOrderStop {
  index: number;
  name: string;
  description: string;
}

interface StateExampleData {
  title: string;                  // #state-title: "Empty", "Filled", "Error", "Disabled"
  componentVariant: Record<string, string>; // Figma properties to set on the linked instance
  textOverrides?: Record<string, string>;   // Realistic content to apply inside the instance
  annotations: AnnotationData[];  // Visual callouts in Example of state with annotations
  optionalDescription?: string;   // Goes in #optional-description
}

interface AnnotationData {
  target: string;                 // What the annotation points to
  output?: string;                // Accessible name/value/output
  role?: string;                  // Semantic role
  state?: string;                 // Exposed state
  requirement?: string;           // Requirement not covered by other fields
}
```

### Structure Rules

| Field | Rule |
|-------|------|
| `componentName` | Type: "Button", "Tooltip", "Tab bar", "Text field", etc. |
| `summary` | Required primary layer. Must be concise, plain language, and scannable before any technical table. |
| `compSetNodeId` | Figma node ID of the exported component set in Main component showcase, from the extraction script. Used for creating linked example roots. |
| `exampleSourceNodeId` | Figma node ID of the exported component or variant used as the top-level example root. Must not point to a primitive. |
| `rootSize` | `{ w, h }` of the default variant. Used to center the component instance in example frames. |
| `elements` | Array of direct children with bounding boxes from extraction. Use this to place annotations on the correct target. |
| `guidelines` | Bullet points. First bullet should describe focus order for compound components. Cover: edge cases, platform differences, focus behavior. |
| `focusOrder` | **Top-level, optional.** Only for compound components (2+ focusable/announced parts). Summarize in the summary or technical appendix; do not force platform tables. |
| `focusOrder.title` | Always `"Focus order"` |
| `focusOrder.stops` | One item per step: `index` is the step number, `name` is the element name, `description` explains the stop |
| `examples` | Required primary layer. One entry per duplicated `#state-template`. |
| `examples.title` | State or archetype title: "Empty", "Filled", "Error", "Disabled", "Selected", "Trailing action". |
| `examples.componentVariant` | Figma properties to set on the linked instance. Use actual available properties. |
| `examples.textOverrides` | Realistic labels, values, placeholders, error messages, helper text, etc. |
| `examples.annotations` | One or more contextual callouts using Output / Role / State / Requirement. |
| `examples.optionalDescription` | Generic implementation details for that state. Keep it line-broken when list-like. |
| `technicalAppendix` | Optional generic details that apply across examples; belongs in the lower Technical appendix block. |

### Annotation formatting

Every visual callout in `Example of state with annotations` uses only populated fields, one per line:

```
Output: [accessible name/value/output]
Role: [semantic role]
State: [programmatic state]
Requirement: [other behavior or constraint]
```

Do not write annotation content as dense paragraphs. Split independent requirements into separate annotations or separate lines.

### Archetype Strategy

For grouped controls (tab bar, radio group), don't document every item. Document representative archetypes:
- "Selected item" + "Unselected item" covers most cases
- Add "Disabled item" only if behavior differs
- Use actual content from the image for realistic examples

---

## Applying the Principles

| If you see... | Merge analysis | Focus stops | Result |
|---------------|---------------|-------------|--------|
| Simple button | Label merges into button | 1 stop: button | Annotate default plus disabled/loading/error only when those states exist |
| Checkbox with label | Label merges into checkbox | 1 stop: checkbox | Annotate unchecked, checked, disabled, error if supported |
| Text field / select field | Label + value + hint merge into field | 1 stop: field (+ trailing action if interactive = 2 stops) | Annotate empty, filled, error, disabled, and trailing action when present |
| Chip with close | Label merges into chip body | 2 stops: chip, close button | Annotate chip body and close action separately |
| Tab bar | — | Tablist + tabs | Annotate selected and unselected tab archetypes, plus disabled if supported |
| List item | Title + subtitle merge into item | 1 stop: list item (+ trailing action if present = 2 stops) | Annotate list item output and trailing action separately when present |
| Tooltip | Bubble is descriptive, not a normal focus stop | 1 stop: trigger | Annotate trigger output and requirement for bubble announcement |
| Card | Heading + description may merge into card | Card link + each action button | Annotate card output and each action archetype |
| State adds error/status content | Error/status updates announcement | Focus stops may stay same | Add a state example showing how the error/status is exposed |

---

## Edge Cases

| Situation | Action |
|-----------|--------|
| Label merges into input | Do NOT annotate the label as a separate focus stop. Include it in the field `Output`. |
| Platform merge behavior differs | Note differences in `#optional-description` or Technical appendix. |
| Element is a live region | Do NOT list it as a focus stop. Add an annotation or optional description explaining announcement behavior. |
| Decorative element | Do not annotate unless decorative treatment is ambiguous. |
| Focus order changes by state | Add a state example showing the changed order. |
| Simple component with no compound parts | Omit `focusOrder`; use state examples for meaningful state changes. |
| Merged parent with one breakout child | If a container uses `mergeDescendants` but one child is independently interactive, list only the interactive child as a focus stop — the container is not a stop |
| Ambiguous merge across platforms | If iOS merges parts but Web keeps them as separate focusable elements, document the superset in `focusOrder` and note platform differences in guidelines |

---

## Common Mistakes

- **Placeholders:** Never use `<label>`; use actual text
- **Curly quotes:** `""` should be `\"`
- **Dense annotations:** Split list-like content into line-broken Output / Role / State / Requirement fields.
- **Missing states:** Document all states
- **Vague guidelines:** Give implementation advice, not description
- **No citations:** Omit `:contentReference`, `oaicite`, etc.
- **Over-grouping:** Not every visual cluster needs a container
- **Under-grouping:** Mutual-selection items need container semantics
- **Missing annotation target:** Every annotation must point to the element or state it describes.
- **Inconsistent role:** If the role is the same across states, keep the wording consistent in each relevant example.
- **Listing merged parts as focus stops:** Label, hint text, and other non-interactive parts that merge into an interactive element are NOT focus stops — do not give them their own entry in `focusOrder`
- **Missing focus order:** Components with 2+ actual focus stops need a focus-order note in summary, optional description, or appendix.
- **Missing state examples:** Error, disabled, selected, expanded, loading, and filled/empty states need examples when they change accessibility behavior.
- **Confusing visual parts with focus stops:** Run the merge analysis before listing focus stops. A text field has 3 visual parts but typically 1 focus stop (the input)
- **Technical-first summary:** Do not start with platform property dumps. The first readable block must be a concise designer summary.

---

## Pre-Output Validation Checklist

Before rendering in Figma, verify your structured data against these checks:

| Check | What to Verify |
|-------|----------------|
| ☐ **Merge analysis done** | Every visual part classified: focus stop, merged into parent, live region, or decorative |
| ☐ **Focus stops only** | `focusOrder` entries are only actual focus stops (interactive elements, navigation containers) — no merged parts listed as separate entries |
| ☐ **Focus order is explained** | If component has 2+ focus stops, focus order is explained in summary, optional description, or appendix |
| ☐ **Focus order omitted when 1 stop** | Simple components with 1 focus stop do NOT include `focusOrder` |
| ☐ **State examples duplicated** | Every relevant state or archetype has its own duplicated `#state-template` |
| ☐ **Exported roots used** | Every `Example of state with annotations` has a top-level instance from Main component showcase, not Primitives or Main Primitive wrapper |
| ☐ **Linked instances used** | Every example root is a linked instance from an exported main component or variant |
| ☐ **Instance configured** | Properties and text overrides make the example state realistic |
| ☐ **Annotations contextual** | Output / Role / State / Requirement callouts point to the exact element or state they describe |
| ☐ **Shared tooltip used** | Every visual annotation uses `Specs-TootlipBody`, not a custom frame or drawn box |
| ☐ **Routed arrows used** | Every connector is a vector arrow with the arrowhead on the target element end |
| ☐ **Line breaks used** | List-like annotation and optional-description content uses line breaks, not dense paragraphs |
| ☐ **Merged parts documented** | Parts that merge are documented in `Output` or optional description on the focus stop they belong to |
| ☐ **All relevant states documented** | Error, disabled, selected, expanded, loading, filled/empty states are represented when accessibility behavior differs |
| ☐ **Guidelines describe merging** | For compound components, guidelines explain what merges and what the user actually lands on |
| ☐ **Straight quotes** | JSON uses ASCII `"` not curly quotes `""` |
| ☐ **No placeholders** | All values use actual text from the component, not `<label>` |
| ☐ **Technical appendix secondary** | Generic platform details are below the contextual examples |
| ☐ **`elements` populated** | `elements` array has entries from extraction for annotation placement when Figma link is provided |

---

## Examples

Use examples to verify the output model is concrete enough. State-rich components should not collapse to one generic example.

### State-rich select/input pattern

For a component like `InputSelect`, generate multiple `#state-template` blocks:

1. **Empty**
   - Instance: placeholder state.
   - Annotation:
     ```
     Output: [Field purpose]. [Placeholder]
     Role: Select field
     Requirement: Placeholder is announced with the field purpose.
     ```

2. **Filled**
   - Instance: selected value state with realistic selected text.
   - Annotation:
     ```
     Output: [Field purpose]. Current value: [Selected value]
     Role: Select field
     ```

3. **Error**
   - Instance: error state with realistic error text.
   - Annotation:
     ```
     Output: [Field purpose]. Current value: [Selected value or placeholder]. [Error message]
     State: Invalid
     Requirement: Error text is associated with the field.
     ```

4. **Disabled**
   - Instance: disabled state.
   - Annotation:
     ```
     Role: Select field
     State: Disabled
     Requirement: Disabled state is exposed programmatically, not only visually.
     ```

5. **Trailing action** (only if the trailing slot is interactive)
   - Instance: field with visible trailing action.
   - Annotation on the field:
     ```
     Output: [Field purpose]. Current value: [Selected value]
     Role: Select field
     ```
   - Annotation on the trailing action:
     ```
     Output: [Action name]
     Role: Button
     Requirement: Action is a separate focus stop.
     ```

Put shared implementation notes in each `#optional-description` or the Technical appendix. Do not put all of this in one paragraph.

### Other patterns

Use one representative example per pattern when states do not change accessibility behavior:
- single-stop component
- compound component with merged parts
- grouped navigation control

If you need deeper implementation examples, consult:
- `screenreader.md` for analysis patterns
- `voiceover.md`, `talkback.md`, and `aria.md` for platform detail
