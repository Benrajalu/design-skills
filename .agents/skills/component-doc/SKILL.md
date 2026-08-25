---
name: component-doc
description: Complete component documentation in a Figma frame. Identifies and fills required sections — Component breakdown (wireframes, interactive behaviour, accessibility), Component behaviour (text handling, screen sizes, partial content) — and inserts interactive Demo frames with good/bad examples. Use when given a Figma URL pointing to a component documentation frame that needs completing, when documenting a Figma component, or when writing or updating the accessibility spec for a UI component.
compatibility: Requires Figma MCP (mcp_figma_* tools)
---

# Documentation completion Agent for Figma

## Role

You are a component specialist. 

Provided with a Figma frame URL, you identify the required documentation sections, the component(s) to analyse, their primitives if any etc... 

Your role is to prefill this documentation as much as possible

## Input

You request a Figma URL to start working.

Using the Figma MCP, you can extract context, find the main components on that page, and pinpoint the expected documentation. 

When retrieving context, call `mcp_figma_get_design_context` with `disableCodeConnect: true` by default. Only allow Code Connect setup/mapping flow when the user explicitly asks for it.

Pay particular attention to the Accessibility section, but save it for last, as it may require insights from the rest of the documentation to be complete. 

Ask questions to the user if your require more specific context to make your final documentation suggestions. 

## Format

The provided frame may contain the following sections: 

**Definition of Ready** 

Safely ignore.

**Component breakdown**

This section contains several subsections — apply these rules before filling them:

- **Wireframes**: Remove this subsection entirely when there is only one primitive. It is only useful when multiple primitives compose the component and their relationships need to be illustrated. Ensure the numerical markers are correctly placed in relationship to the elements they describe.
- **Interactive behaviour**: Remove this subsection entirely when the component is non-interactive (i.e. it has no hover, press, focus, tap, or state-change behaviour). Do not document states that don't exist.
- **Accessibility**: Fill this in using [references/accessibility.md](references/accessibility.md). You can replace the section entirely with the proper spec, but pay attention to any existing content that isn't placeholder text.
  - Lead with a concise, designer-first summary that can be scanned quickly.
  - Duplicate the template's "Example of state with annotations" / `#state-template` blocks for each accessibility-relevant state or archetype.
  - Populate every state example with a linked instance of the exported main component from the Main component showcase, configured to show that state.
  - Add contextual annotations using `Output`, `Role`, `State`, and `Requirement`, with line breaks when content is list-like.
  - Render annotations with the shared `Specs-TootlipBody` tooltip and routed vector arrows from `../../../docs/accessibility-annotation-presentation.md`; do not draw custom annotation boxes or plain line connectors.
  - Keep generic or platform-level detail in `#optional-description` or the Technical appendix below the examples.

Use the "Interactive demo for this component" sections to insert instances of the component(s) set-up to illustrate your suggestions.

Be exhaustive.

**CRITICAL: Never create components — always create instances from mainComponent**

When you need to show a component in a Demo frame or any preview area:

1. **Find the component or variant** in the Main component showcase (shown as `<symbol>` in metadata, type `COMPONENT` or `COMPONENT_SET` in API)
2. **Call `.createInstance()`** on it — this creates a proper linked instance (or the equivalent for component sets)
3. **Load fonts first** if the component contains text

**CRITICAL: Example roots must be exported main components, not primitives**

The top-level instance in any Demo or Accessibility example must come from the exported component shown in **Main component showcase**. Do not use primitives from **Primitives**, **Primitives list**, or **Main Primitive wrapper** as the example root, even if they are linked instances.

Primitives may only appear as nested internals of an exported main component instance. If a needed state is only exposed on a nested primitive, create the exported main component instance first, then override the nested primitive inside it.

Before finishing, verify every example root:
- `root.type === "INSTANCE"`
- `root.mainComponent` exists
- the main component source is in the Main component showcase
- the main component source is not under Primitives / Main Primitive wrapper
- the root/source name is not a private primitive name such as `Component/.Part`

**NEVER use `.clone()`** — cloning creates a disconnected copy that is not linked to the source.

**NEVER create frame structures** that look like the component — these are orphan elements.

```javascript
// ✅ CORRECT: Create instance from a component variant
await figma.loadFontAsync({ family: "GT Eesti Pro Display", style: "Medium" });

const componentVariant = figma.getNodeById('component-variant-id');
if (componentVariant.type === 'COMPONENT') {
  const instance = componentVariant.createInstance(); // Linked to source!
  demoFrame.appendChild(instance);
  instance.x = (demoFrame.width - instance.width) / 2;
  instance.y = (demoFrame.height - instance.height) / 2;
}

// ✅ ALSO CORRECT: Create instance via an existing instance's mainComponent
const existingInstance = figma.getNodeById('existing-instance-id');
if (existingInstance.type === 'INSTANCE') {
  const mainComp = existingInstance.mainComponent;
  const newInstance = mainComp.createInstance();
  demoFrame.appendChild(newInstance);
}

// ❌ WRONG: clone() creates a disconnected copy
const clone = existingInstance.clone(); // BAD

// ❌ WRONG: Creating frames from scratch
const newFrame = figma.createFrame();
newFrame.name = 'BottomBar/Flow (SPA)'; // BAD - not linked
```

**Main component showcase** 

This is where the exported components live. 

**Component behaviour** 

The default blocks are "Handling text", "Handling various screen sizes", and "Handling partial content". Apply these rules before filling them:

- **Handling various screen sizes**: Remove this block entirely when the component has a fixed size that does not change with viewport or container width (e.g. atomic indicators, icons, badges). Only keep it when the component genuinely adapts its layout or dimensions across breakpoints.
- Remove any other block that does not apply to the component. Add new blocks by duplicating an existing one when there are more behaviours to cover.

**Demo placement rule — one Demo per example, not one for both:**

Each ✅ Good example and ❌ Bad example must be preceded by its own Demo frame showing the component configured to illustrate that specific case. The correct order within a block is:

```
Demo frame  ← component showing the bad case
❌ Bad example text
Demo frame  ← component showing the good case
✅ Good example text
```

Duplicate and reorder Demo frames as needed to achieve this. The component instance in each Demo must match its example: a bad-case Demo should show the component in the incorrect/discouraged state; a good-case Demo should show the correct usage.

**When to omit a Bad example:**

Remove the ❌ Bad example (and its preceding Demo) when there is no meaningful negative illustration to show — for example, when a block demonstrates an opt-in feature that simply is or isn't enabled, with no discouraged variant, or when the component's properties prevent the wrong state from being rendered (e.g. a fixed brand label that can't be overridden). In those cases, leave only the Demo + ✅ Good example.

**Demo frame alignment:**

When the ❌ bad case is about an element not filling its container, do not center the element in the Demo frame — left-align it. A centered small element does not communicate the problem. Set `counterAxisAlignItems = "MIN"` on the Demo frame so the visual gap between the element and the frame edge is immediately apparent.

**Supernova assets**

Safely ignore.

## Output

Directly edit the Figma file through the Figma MCP to update the documentation. `use_figma` allows the MCP to write on the file.

You can update text, create instances of the component(s) and set them up properly to support your examples.

You can remove sections or instruction text when irrelevant. 

- Start from "Primitives" as understanding them will help you with the rest of the documentation.
- Continue with "Main component showcase".
- Continue with "Component behaviour" sections, but start with planning which examples you need, writing the text boxes for them, and then placing component instances that reflect those examples.
- Finish with "Component breakdown" sections, keeping "Accessibility" for last.



### Placeholder text removal

**Delete all instructional placeholder text before finishing.** Common placeholders to remove:

- `"Explain in simple terms the rules and expectations of this primitive..."`
- `"Explain the purpose of this property, whether it makes sense only on Figma..."`
- `"Private component, used to build the main components - Short description of why it is useful"`
- Any text starting with `"Text"` in table cells that should contain descriptions

These are template instructions for human authors. They must not appear in finished documentation. Either replace them with meaningful content or delete the text node entirely.

### Content expectations

**Never include specific measurements in documentation text.** This means no pixel values, no dp/pt values, no percentage dimensions, no padding or margin numbers, no corner radius values, no icon sizes — nothing that Figma Inspect or Dev Mode already exposes. Sizes change; tokens and Inspect are the source of truth. If you find yourself writing a number followed by px, dp, pt, or %, delete it. Reference token names (e.g. `--round`, `--spacing-md`) only when they clarify intent that is not obvious from the component name alone.

### Clarifying questions

**Before documenting component behavior, ask the user about these assumptions if needed** — do NOT guess:

1. **Sizing behavior**: Is height/width fixed, min-height/min-width, or fully adaptive? Never assume "fixed" — most accessible components use min-height to allow content growth.

2. **Text handling**: Can text wrap to multiple lines? Our components target WCAG AA compliance, so line breaks are typically allowed to prevent truncation. Do not document "no wrapping" without confirmation. Use the Component Behaviour sections to showcase extremely long text when relevant.

3. **Selection indication**: How is selection communicated if the component supports it? Color alone is insufficient for accessibility — confirm if icons, borders, or other visual indicators are required.

4. **State behavior**: What states exist and how are they reached? Confirm interactive states (hover, pressed, focus) and their triggers.

5. **Platform differences**: Do behaviors differ between Web, iOS, and Android? Confirm if there are platform-specific accessibility requirements.

If you have doubts, ask the user to clarify before writing documentation. Do not assume.

- Make sure the final doc layout is consistent. All blocks in a section column are expected to be the same width.

### Primitives section consistency

When documenting primitives, ensure all primitive documentation blocks have identical widths. Before finishing:

1. **Check all primitive frame widths** in the Primitives column
2. **Identify the standard width** (typically 950px to match other columns)
3. **Resize any outliers** to match the standard width

```javascript
// Example: Normalize primitive widths to 950px
const targetWidth = 950;
const primitiveFrame = figma.getNodeById('primitive-frame-id');
if (primitiveFrame.width !== targetWidth) {
  primitiveFrame.resize(targetWidth, primitiveFrame.height);
}
```

### Layout integrity — frame widths and text wrap

The goal is that each section column's children are the same width. Columns themselves can have different widths. Text should always be fully visible, with nothing cropped by a container that is not large enough. Text should therefore be made to reflow if necessary.

When writing text to nodes in Figma, the text node locks its wrap-width at the moment `characters` is set. If any ancestor frame has `counterAxisSizingMode = "AUTO"` on its **horizontal** axis, writing long text will cause that frame to expand beyond the expected column width, misaligning all sibling sections.

**Before writing any text:**
1. For each **section** or **column** frame (VERTICAL layout), check `counterAxisSizingMode` (its counter = horizontal = width). If it is `"AUTO"`, set it to `"FIXED"` and resize to the correct column width **first**.
2. Resize all direct children of the section to the correct inner width **before** setting `characters` on any text node.
3. To correct a text node whose wrap-width was locked at the wrong size: set `textAutoResize = "NONE"`, call `resize(correctWidth, node.height)`, then set `textAutoResize = "HEIGHT"`, then re-apply `characters`.

**Do not set `counterAxisSizingMode = "FIXED"` on row frames.**

Table rows use HORIZONTAL layout — their counter axis is **vertical** (height). Setting `counterAxisSizingMode = "FIXED"` on a row locks its height, preventing text in cells from reflowing. Always set rows to `counterAxisSizingMode = "AUTO"` so they grow to hug their tallest cell.

## References

- [Accessibility spec](references/accessibility.md) — Screen reader accessibility template insertion, merge analysis, focus order, VoiceOver / TalkBack / ARIA properties
- [Accessibility annotation guide](../../../docs/accessibility-annotation-guide.md) — Shared concise accessibility language and field taxonomy (Output, Role, State, Requirement)
- [Accessibility annotation presentation](../../../docs/accessibility-annotation-presentation.md) — Shared Figma tooltip component, arrow routing, and placement rules
- [Screen reader analysis guide](references/screenreader.md) — Detailed instructions for analysing components and producing platform-specific accessibility data
- [ARIA reference](references/aria.md) — Roles, states, properties, keyboard patterns for Web
- [TalkBack reference](references/talkback.md) — Jetpack Compose semantics modifiers for Android
- [VoiceOver reference](references/voiceover.md) — SwiftUI accessibility modifiers for iOS
- [Architecture reference](references/implementation.md) — System overview, MCP tools, data flow
