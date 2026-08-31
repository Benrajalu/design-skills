# Component behaviour

Component behaviour documents how authors should use the public component in realistic situations. It is not a requirement to fill the starter template blocks.

Use this reference before writing or editing the **Component behaviour** column.

## Goal

Behaviour examples should teach the highest-signal usage rules for the component.

Good behaviour documentation:
- is grounded in the public component's actual properties, variants, slots, and states
- shows realistic product content
- uses linked instances from **Main component showcase**
- makes the Demo frame prove the adjacent Good or Bad example text
- removes template blocks that do not apply
- adds or renames blocks when a component-specific pitfall is more useful than the starter headings

## Workflow

### 1. Build a behaviour inventory

Before writing example text, inspect the exported main component and note:

- public variants and properties
- text properties and editable text layers
- slots and instance-swap properties
- state variants such as default, hover, pressed, focus, selected, disabled, loading, error, expanded, and collapsed
- layout variants such as compact, wide, stacked, density, and platform
- whether the component has one focus stop or multiple focus stops
- which nested primitives explain behaviour, without using them as top-level Demo roots

This inventory is analysis only. Do not paste it into the documentation.

### 2. Select behaviour scenarios

Choose scenarios from the component's actual risk areas. Keep the starter blocks only when they are useful.

Keep or add scenarios when they explain:

| Scenario type | Use when |
|---|---|
| Text handling | Labels, values, placeholders, helper text, or error text can wrap, truncate, overflow, become ambiguous, or need realistic content. |
| Responsive layout | The component changes composition, layout, density, or child relationship across available space. |
| Partial content | Optional slots, missing values, helper content, metadata, icons, or actions change the component's meaning. |
| State transitions | Focus, hover, pressed, selected, expanded, collapsed, loading, disabled, or error states change how the component should be used. |
| Compound interaction | The component contains more than one meaningful target, such as a selector plus input, field plus action, menu item plus trailing control, or card plus nested CTA. |
| Validation and recovery | Invalid content, required input, helper/error relationship, or recovery flow is important. |
| Selection communication | Selected/current/checked state must not rely on colour alone. |
| Authoring constraints | Two or more properties must be changed together for the component to make sense. |

Remove starter blocks that do not apply. For example, remove **Handling various screen sizes** when the component is fixed and does not adapt its layout or composition.

### 3. Plan each Demo before writing example text

For every Good or Bad example, define a demo spec:

- exported component variant or main component source
- public property overrides
- text overrides
- slot or instance-swap overrides
- nested primitive overrides only when the public component does not expose the state directly
- what visible behaviour the Demo is proving

If the intended example cannot be represented with a linked exported main-component instance, change or remove the example.

### 4. Write example text to match the Demo

Each Demo and example text pair must pass this test:

- The Demo visibly illustrates the adjacent text.
- A Bad Demo shows the discouraged situation, not a generic default.
- A Good Demo shows the recommended alternative, not the same instance with unchanged content.
- The text names the rule being taught, not the mechanics of the Figma setup.
- The content is realistic for product usage.

Do not write example text first and then insert a generic Demo. The Demo spec comes first.

## Pitfall checklist

Ask "what can go wrong with this component?" before deciding that the starter blocks are enough.

Check for:

- overly long, localized, missing, duplicate, or ambiguous content
- placeholder text used as the only instruction
- label, value, hint, helper, and error text that do not belong together
- state conflicts such as disabled plus error, selected plus unavailable, loading plus value
- focus ambiguity in compound controls
- separate actions that look like part of the same target
- optional slots that are hidden but change meaning when present
- layout choices that weaken grouping or reading order
- visual-only state indication
- icons or badges whose meaning is unclear without text
- properties that must be changed together
- bad states that are impossible because the component API prevents them

Prefer strong, component-specific scenarios over weak template-shaped examples. Two or three scenarios may be enough for a simple component, but this is not a cap: state-rich, compound, or validation-heavy components should document every meaningful behaviour risk.

## Demo requirements

Every Demo in Component behaviour must:

- use a linked instance of an exported main component from **Main component showcase**
- never use a primitive as the top-level Demo root
- be configured with actual public properties and realistic text
- match the Good or Bad example immediately below it
- be duplicated so each Good and Bad example has its own Demo
- be removed when its adjacent example is removed

Nested primitive overrides are allowed only when:

1. the top-level Demo root is still the exported main component, and
2. the public component does not expose the needed state directly.

## Demo frame sizing and layout

Do not let the starter Demo frame size constrain the example.

Default to a centered auto-layout Demo frame:
- set the Demo frame to vertical auto layout
- keep the Demo frame width aligned with the section column
- center children on both axes
- resize the Demo height or use vertical hug sizing so the component has enough breathing room
- keep `clipsContent = false` unless the example is intentionally about clipping

Resize Demo frames whenever the scenario needs more room, especially for:
- error states with helper or validation text
- focus states with visible outlines
- narrow or stacked variants
- multiple instances in one Demo
- before/after or state-comparison examples
- components with long realistic content

Use manual layout only when spatial relationships are the point of the example:
- constrained parent or clipping demonstrations
- left-aligned bad examples where centering would hide the issue
- side-by-side comparisons
- examples that need labels, containers, or other context around the component

When using manual layout, document the spatial setup visually and still resize the Demo frame so nothing feels cramped unless the cramped layout is the explicit bad example.

## Good and Bad examples

Use a Bad example only when it teaches a real decision.

Good Bad examples:
- show a common authoring mistake
- are possible to render with the component
- contrast clearly with the Good example
- make the risk visible in the Demo

Remove the Bad example when:
- the component API prevents the bad state
- the bad state is purely hypothetical
- the negative version would be indistinguishable from the good one
- the block documents an opt-in feature with no discouraged alternative

When removing a Bad example, remove its Demo too.

## Block naming

Rename blocks when the starter heading is too generic.

Examples:
- `Handling responsive layout` instead of `Handling various screen sizes`
- `Choosing focus target` for a compound control
- `Recovering from error` for validation-heavy fields
- `Using optional actions` for components with trailing actions
- `Communicating selection` for selected/current states

The heading should describe the scenario, not the template slot.

## Validation checklist

Before finishing Component behaviour:

| Check | What to verify |
|---|---|
| Scenario relevance | Each block documents a real component behaviour or pitfall. |
| Template pruning | Starter blocks that do not apply are removed or renamed. |
| Demo first | Every example has an explicit Demo configuration behind it. |
| Linked roots | Every Demo root is a linked exported main-component instance. |
| No primitive roots | No Demo root comes from Primitives, Primitives list, or Main Primitive wrapper. |
| Demo/text match | Every Demo visibly proves the adjacent Good or Bad text. |
| Bad example validity | Bad examples are possible, meaningful, and visually distinct. |
| Impossible bad states | Bad examples prevented by the component API are removed. |
| Realistic content | Text and values use product-realistic examples, not placeholder-only defaults. |
| One Demo per example | Each Good and each Bad example has its own Demo immediately before it — this is about not sharing one Demo between Bad and Good, not about limiting instance count. A single Demo may hold multiple instances/configs when that better proves its example. |
| Demo sizing | Demo frames are resized or set to hug content when the starter height is too small. |
| Demo layout | Normal demos use centered auto layout; manual layout is reserved for spatial storytelling. |
| Layout clarity | Bad demos about sizing or alignment are not centered in a way that hides the issue. |
