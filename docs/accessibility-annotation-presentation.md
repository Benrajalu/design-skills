# Accessibility Annotation Presentation

Shared Figma presentation contract for accessibility annotations. Use this when a skill draws accessibility callouts in Figma.

This reference covers **how annotations look and connect**. Use `accessibility-annotation-guide.md` for **what to annotate** and how to write `Output`, `Role`, `State`, and `Requirement`.

## Core rules

- Use the shared tooltip component. Do not draw custom annotation boxes.
- Build content from populated fields only: `Output`, `Role`, `State`, `Requirement`.
- Put one field per line. Never collapse list-like content into one paragraph.
- Draw arrows as vectors using the routed arrow helpers below. Do not use `figma.createLine()` for production annotations.
- The arrow endpoint is a contract: every populated field must describe the exact element at the endpoint.
- Use linked component instances for component examples. Do not draw fake components.

## Tooltip component

The annotation tooltip is `Specs-TootlipBody` from the Specs components library.

| Property | Value |
|---|---|
| Component name | `Specs-TootlipBody` |
| Component key | `5d9da1afbffd7efd927d6b59c9d969b34286594b` |
| Source file key | `rPl6NXhJ9nU0b7xXfr4jBI` |
| Source node ID | `4002:2156` |
| Background colour | `#d12771` - `{ r: 0.8196, g: 0.1529, b: 0.4431 }` |

Use `setProperties()` on the imported component instance. Do not recreate the tooltip manually.

```javascript
const TOOLTIP_KEY = '5d9da1afbffd7efd927d6b59c9d969b34286594b';
const TIP_COLOR = { r: 0.8196, g: 0.1529, b: 0.4431 };

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

const tooltipComponent = await figma.importComponentByKeyAsync(TOOLTIP_KEY);
const tip = tooltipComponent.createInstance();
tip.setProperties({
  'Title#4002:0': 'Passenger selector',
  'Content#4002:1': buildTooltipContent({
    output: 'Set passengers. Current value: 3 passengers',
    role: 'Button'
  }),
  'show_title#4002:2': true
});
```

| Property key | Type | Role |
|---|---|---|
| `Title#4002:0` | TEXT | Short target label naming what the arrow points to |
| `Content#4002:1` | TEXT | Populated annotation fields |
| `show_title#4002:2` | BOOLEAN | Show or hide the title row |

## Target ownership

Every populated field must be true for the element at the arrow endpoint.

| If the field describes... | Then point the arrow at... |
|---|---|
| a whole single-stop control | the component surface |
| a selected, invalid, disabled, expanded, or loading control | the component surface showing that state |
| a helper or error message associated with a field | the field surface if the message is part of that field's output |
| a separate trailing action | the trailing action itself, not the field |
| a group-level requirement | the group container or a clear representative with catch-all wording |

If one tooltip would need to point at two targets, split it into two tooltips or rewrite it as a generic note outside the visual annotation.

## Drawing inside component-doc state examples

`component-doc` places annotations inside the Accessibility template's `Example of state with annotations` frame, not in a page-level side container.

Use this simplified in-frame layout:

1. Set the example frame to `layoutMode = "NONE"` and `clipsContent = false`.
2. Add the linked component instance first.
3. Add one or more `Specs-TootlipBody` instances.
4. Place tooltips to the right of the component when space allows.
5. Draw routed vector arrows from tooltip edge to the target element.
6. If a tooltip grows after `setProperties()`, read its actual size before drawing the arrow.

## Routed arrow helpers

Figma normalises vector vertex coordinates to be non-negative. Always compute path points in canvas or parent-local coordinates first, find the bounding-box minimum, then express vertices as offsets from that minimum.

```javascript
const STRAIGHT_THR = 5;

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

function arrowH(tipEdgeX, tipCY, elEdgeX, elCY, gutterX) {
  const pts = Math.abs(tipCY - elCY) <= STRAIGHT_THR
    ? [{ x: tipEdgeX, y: elCY }, { x: elEdgeX, y: elCY }]
    : [
        { x: tipEdgeX, y: tipCY },
        { x: gutterX, y: tipCY },
        { x: gutterX, y: elCY },
        { x: elEdgeX, y: elCY }
      ];
  return buildArrow(pts);
}

function arrowV(tipCX, tipEdgeY, elCX, elEdgeY, gutterY) {
  const pts = Math.abs(tipCX - elCX) <= STRAIGHT_THR
    ? [{ x: elCX, y: tipEdgeY }, { x: elCX, y: elEdgeY }]
    : [
        { x: tipCX, y: tipEdgeY },
        { x: tipCX, y: gutterY },
        { x: elCX, y: gutterY },
        { x: elCX, y: elEdgeY }
      ];
  return buildArrow(pts);
}

async function drawArrow(parent, arrow) {
  const vec = figma.createVector();
  parent.appendChild(vec);
  vec.name = 'Annotation arrow';
  await vec.setVectorNetworkAsync(arrow.network);
  vec.x = arrow.vecX;
  vec.y = arrow.vecY;
  vec.strokes = [{ type: 'SOLID', color: TIP_COLOR }];
  vec.strokeWeight = 2.5;
  vec.fills = [];
  return vec;
}
```

For right-side in-frame annotations:

```javascript
const tip = tooltipComponent.createInstance();
exampleFrame.appendChild(tip);
tip.x = component.x + component.width + 80;
tip.y = component.y + 8;
tip.setProperties({
  'Title#4002:0': ann.label,
  'Content#4002:1': buildTooltipContent(ann),
  'show_title#4002:2': true
});

const arrow = arrowH(
  tip.x,
  tip.y + tip.height / 2,
  component.x + component.width,
  component.y + component.height / 2,
  component.x + component.width + 40
);
await drawArrow(exampleFrame, arrow);
```

## Validation checklist

Before finishing a Figma annotation pass:

- No hand-drawn annotation boxes remain.
- Every annotation tooltip is an instance of `Specs-TootlipBody`.
- Tooltip content uses one populated field per line.
- Arrows are vectors with arrowheads on the target element end.
- No arrow points at whitespace or at a parent when the content describes a child.
- No annotation contains platform API syntax.
- No tooltip overlaps the target component or another tooltip.
- The example frame has `clipsContent = false` if arrows extend beyond a tooltip body.

## Known gotchas

| Situation | Fix |
|---|---|
| Tooltip content changes size after setting properties | Read `tip.width` and `tip.height` after `setProperties()` before drawing arrows. |
| Arrowhead points away from the element | Ensure the target element is the last vertex and receives `strokeCap: 'ARROW_LINES'`. |
| Arrow lands in the wrong place | Use `buildArrow(pts)` so all vector vertices are non-negative offsets from the path bounding box. |
| Reparented annotations jump | Record canvas coordinates before reparenting, then subtract the new parent origin after append. |
| A label/value override wraps unexpectedly | Resize the text node before or immediately after setting `characters`, or use shorter realistic copy. |
