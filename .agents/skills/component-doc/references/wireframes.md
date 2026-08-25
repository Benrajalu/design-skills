# Wireframes

Wireframes explain component anatomy through visible, inspectable examples. They are not a layer inventory.

## When to keep or remove

Remove the Wireframes subsection when the public component has one meaningful part and no optional slots, state-dependent anatomy, or nested primitives worth explaining.

Keep Wireframes when:
- the public component is composed of multiple meaningful primitives
- optional leading, trailing, helper, status, or action slots exist
- different public configurations reveal different anatomy
- a private primitive is important for maintainers to understand

## Source rules

Every wireframe example root must be a linked instance of an exported component from **Main component showcase**.

Do not use private primitives from **Primitives**, **Primitives list**, or **Main Primitive wrapper** as the top-level example root, even when they are linked instances.

Primitives may be referenced when they are visible nested internals of the exported instance being shown. If a primitive is hidden in the current configuration, do not mark it there.

If a hidden or optional primitive is important to document, add another visible configuration of the exported main component. For example:
- duplicate the wireframe example and change the public variant/property
- add a second exported main-component instance inside the same example frame
- override a nested primitive inside the exported main component only when the public component does not expose that state directly

## Visible-target rule

A numbered marker must point to something visible in the same wireframe example.

Do not place markers for:
- hidden layers
- inactive variant branches
- absent optional slots
- primitives that exist only in the layer tree but are not visible in the current configuration
- speculative future content

If the target is not visible, either:
1. show a configuration where the target is visible, or
2. remove the marker and remove its matching description row.

## Marker and description contract

Markers and text rows must match exactly.

For every marker:
- there is exactly one row starting with the same number
- the row starts with the exact visible primitive or public child component name the marker points to
- the row explains the part's purpose, not its implementation details

For every row:
- there is exactly one visible marker with that number
- the target exists and is visible in the example

Use the real primitive/component name first, then explain the purpose:
- `1 - InputLabel - Gives the field its visible label.`
- `2 - InputSelect/.Select/Empty - Shows placeholder content before a value is chosen.`
- `3 - InputSelect/.Select/Filled - Shows the selected-value surface.`

Do not replace primitive names with generic labels like "Empty select surface" or "Right affordance". The wireframe section's purpose is to show the exported component's composition model, so the row must make the link to the primitive name explicit.

Do not list private primitive names only because they appear in the layer tree. Use them only when they clarify how the exported component is built and the primitive is visible in the example. If the primitive is hidden, show another exported component configuration where it is visible or omit it.

## Marker placement

Markers are pointer badges, not loose labels. The non-rounded marker corner is the pointer and must touch a real corner of the target it describes.

A marker is correctly placed when:
- exactly one marker corner is non-rounded
- that non-rounded corner touches one target corner
- the marker sits outside the target's readable content
- the marker is visibly closer to its own target than any other part

Use this placement strategy:

1. Compute the target's bounding box inside the wireframe example.
2. Choose the target corner that gives the clearest pointer location:
   - target top-left: marker bottom-right corner touches it
   - target top-right: marker bottom-left corner touches it
   - target bottom-right: marker top-left corner touches it
   - target bottom-left: marker top-right corner touches it
3. Change the marker corner radii or direction variant so the touching marker corner is the only non-rounded corner.
4. Set the marker position so the non-rounded corner exactly meets the chosen target corner:
   - bottom-right pointer to target top-left: `marker.x = target.x - marker.width`, `marker.y = target.y - marker.height`
   - bottom-left pointer to target top-right: `marker.x = target.x + target.width`, `marker.y = target.y - marker.height`
   - top-left pointer to target bottom-right: `marker.x = target.x + target.width`, `marker.y = target.y + target.height`
   - top-right pointer to target bottom-left: `marker.x = target.x - marker.width`, `marker.y = target.y + target.height`
5. Reject placements that:
   - overlap another marker
   - cover readable text
   - cover important icon detail
   - sit outside the example frame
   - are closer to another documented target than their own target
6. If the first corner collides, try another target corner and update the marker's non-rounded corner to match.
7. If no target corner is safe, add another visible component configuration instead of floating or stacking markers.

For small adjacent targets, prefer placing markers outside the component, around its perimeter. Do not pile multiple dots on the same component corner.

Do not place markers in the middle of a side unless the target has no usable visible corner. If you must use a side-center fallback, document the target with another visible configuration first whenever possible.

## Multiple configurations

Use multiple configurations when anatomy is mutually exclusive or state-dependent.

Examples:
- Empty and filled surfaces should be shown as separate examples if both are documented.
- Error/helper message anatomy should be shown in an error configuration, not described from a default configuration.
- Optional leading/trailing slots should be shown only in a configuration where they are visible.
- Standalone and default variants should not share markers unless the same visible part is present in both.

When multiple configurations are shown in one wireframe block:
- label each configuration clearly
- place markers within or immediately around the configuration they describe
- make description rows mention the configuration when needed

## Validation checklist

Before finishing Wireframes:

| Check | What to verify |
|---|---|
| Exported roots | Each example root is a linked instance from Main component showcase |
| No primitive roots | No top-level example is from Primitives, Primitives list, or Main Primitive wrapper |
| Visible targets | Every marker points to a visible element in the same example |
| No hidden rows | No row describes a hidden, inactive, or absent layer |
| Primitive names | Every row starts with the exact visible primitive or public child component name |
| 1:1 mapping | Marker numbers and description rows match exactly |
| No overlaps | Markers do not overlap each other or critical content |
| Pointer corner | Each marker has exactly one non-rounded corner and that corner touches the target corner |
| Configurations sufficient | Mutually exclusive anatomy is shown through additional visible configurations |
| Text is useful | Rows explain purpose and usage, not raw layer structure alone |
