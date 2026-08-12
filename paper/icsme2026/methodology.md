# Methodology

## III. Shape-Agnostic Focus Obscuration Analysis

### A. Measurement Validity Problem

WCAG 2.4.11 and 2.4.12 require that a focused UI component not be hidden by
author-created content. The normative text frames this in terms of the
*component* — the rendered element as perceived by the user — not its
geometric layout proxy. Any automated approach must therefore measure
obscuration relative to the component's actual visible area.

Existing automated approaches, including our prior tool [NavA11y cite],
approximate the component's area using its axis-aligned bounding box,
obtained via `getBoundingClientRect()`. A uniform 7×7 grid of sample points
is distributed across this box, and `document.elementsFromPoint()` is queried
at each point. A point is classified as obscured if the topmost element in
paint order above the focused element carries a visible background, is a
replaced element (`<img>`, `<video>`, `<canvas>`), or applies a
backdrop-filter. The obscuration ratio is:

$$\text{ratio}_{\text{bbox}} = \frac{|\{p \in G : \text{obscured}(p)\}|}{|G|}$$

where $G$ is the 7×7 grid of points. WCAG 2.4.11 (Level AA) fails when
`ratio ≥ 0.99`; WCAG 2.4.12 (Level AAA) fails when `ratio > 0`.

The flaw is a measurement validity error: the bounding box is not the
component. For non-rectangular elements, the bounding box includes regions
that are visually transparent — regions that belong geometrically to the
background, not to the component. When `elementsFromPoint()` is queried at
these corner points, it returns background elements with opaque backgrounds,
which the algorithm classifies as obscuring content. The element is reported
as partially obscured even when no author-created content overlaps it.

The magnitude of this error scales with the degree of non-rectangularity:

| Shape                      | Bbox coverage by rendered area | Max spurious ratio |
|----------------------------|--------------------------------|--------------------|
| Circle (border-radius: 50%)| ~78.5% (pi/4)                  | ~21.5%             |
| Pill button (r = h/2)      | ~84-92% (width-dependent)      | ~8-16%             |
| Triangular clip-path       | ~50%                           | ~50%               |
| Multi-line link (2 lines)  | varies with line length        | depends on layout  |

Under WCAG 2.4.12 — which fails on any `ratio > 0` — every circular or
heavily rounded focusable element fails unconditionally, regardless of actual
page layout. This is not a measurement noise problem; it is a structural
defect in the measurement model.

---

### B. Shape Boundary Determination

The corrected approach introduces a per-element shape boundary determination
step. For each focused element, the algorithm classifies it into one of four
categories and computes the set of interior points $S \subseteq G$ that lie
within the rendered shape. Points outside $S$ are excluded from both the
numerator and denominator of the obscuration ratio.

**Classification is applied in priority order** to handle elements that
combine multiple shape-affecting properties:

1. **Clip-path** — checked first, as it overrides all other boundary
   definitions. If `clip-path` is not `none`, the clipped shape is the
   boundary regardless of `border-radius`.

2. **Multi-line inline** — if `getClientRects()` returns more than one
   rectangle for the element (indicating line-wrapped layout), the boundary
   is the union of those rectangles. Inter-line gaps and the empty portion of
   short final lines are excluded.

3. **Border-radius** — if any corner has a non-zero computed radius, the
   boundary is a rounded rectangle defined by the four corner arcs.

4. **Default** — otherwise, the full bounding box is used as-is.

**Geometric containment tests per category:**

*Rounded rectangles (including circles):* Each corner arc is an ellipse
quadrant with horizontal radius $r_x$ and vertical radius $r_y$ derived from
the computed `border-radius` values and the element's dimensions. A point
$(x, y)$ passes the containment test for a given corner if, when projected
into the corner's ellipse coordinate system with center $(c_x, c_y)$:

$$\frac{(x - c_x)^2}{r_x^2} + \frac{(y - c_y)^2}{r_y^2} \leq 1$$

A point failing this test for any corner arc is outside the rendered shape
and is skipped. For a full circle, all four corners share the same radii and
center rule; the test degenerates to a single ellipse containment check.

*Multi-line inline:* A point $(x, y)$ is interior if and only if it falls
within at least one rectangle returned by `getClientRects()`. Each rectangle
is an axis-aligned box; containment is a simple range check per fragment.

*Clip-path polygon:* A point $(x, y)$ is interior if it lies within the
clip polygon. We apply the standard ray-casting algorithm: cast a horizontal
ray from $(x, y)$ and count edge crossings with the polygon boundary; an odd
count indicates interior. For `clip-path: circle()` and `clip-path: ellipse()`
functions, the ellipse containment formula above applies directly with the
clip-defined radii and center.

---

### C. Modified Obscuration Measurement

Let $S \subseteq G$ denote the set of grid points that pass the shape
containment test for element $e$. The revised algorithm is:

```text
S ← { (x,y) ∈ G : isPointInShape(e, x, y) }
obscuredShapePoints ← 0

for each (x, y) in S:
    stack ← elementsFromPoint(x, y)       // paint-order, topmost first
    for each topEl in stack:
        if topEl is e or e contains topEl:
            break                          // element is visible here
        if isVisuallyCovering(topEl):
            obscuredShapePoints++
            break                          // point is obscured

obscuredRatio ← obscuredShapePoints / |S|
```

`isVisuallyCovering(el)` returns true when `el` has: opacity > 0.1, and at
least one of: non-transparent background color, replaced content tag
(`IMG`, `VIDEO`, `IFRAME`, `CANVAS`, `SVG`), native input control, or active
`backdrop-filter`. This mirrors the detection logic of the original approach;
only the set of probe points changes.

**Edge cases:**

- *Element fully outside viewport:* If the clipped visible area has zero
  width or height, the element is unreachable by keyboard users and is
  reported as fully obscured (`obscuredRatio = 1.0`) without probing.

- *Degenerate shape (|S| = 0):* If the 7×7 grid resolution is insufficient
  to place any point inside the rendered shape (e.g., a very small rounded
  element), the algorithm falls back to the bounding-box center point.

- *Combined border-radius and clip-path:* Clip-path takes precedence (Step 1
  above); `border-radius` has no additional effect when a `clip-path` is
  active, matching CSS rendering semantics.

**Complexity:** Shape boundary determination adds O(1) geometric computation
per element (one classified containment test per grid point). The dominant
cost remains `elementsFromPoint()`, which is unchanged. Total overhead is
negligible relative to the browser's layout and paint pipeline.

The net effect is a shift in the denominator — from the number of bounding-box
grid points to the number of shape-interior grid points — aligning the
measurement model with the normative intent of WCAG 2.4.11 and 2.4.12: the
question is whether the *component*, as visually rendered, is hidden.
