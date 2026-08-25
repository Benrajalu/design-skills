# A11y Check - Annotation Drawing Reference

> **Purpose:** Complete reference for placing annotation tooltips beside a Figma feature frame and connecting them to their target elements with routed arrows.
>
> **What belongs here:** tooltip component spec, stagger algorithm, arrow routing function, complete ready-to-run code.
>
> **What does NOT belong here:** accessibility criteria logic, which elements to annotate, or how to interpret design content - those belong in SKILL.md.

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
