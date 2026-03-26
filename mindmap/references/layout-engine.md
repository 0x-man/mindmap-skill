# Layout Engine Reference

The mind map supports four layout modes. The engine auto-selects based on content shape,
but the user can force a mode by setting `layout` in `mindmapData` or by asking
("make it a tree", "use flow layout", "switch to radial").

## Auto-Detection Logic

Analyze the extracted content before layout to pick the best mode:

```
if (mindmapData.layout)          → use the specified mode
else if (branches > 6)           → semicircle
else if (contentIsSequential)    → flow
else if (contentIsHierarchical)  → tree
else                             → radial (default)
```

**Sequential signals** — use flow layout when:
- Content describes a process, pipeline, or workflow
- Branches have a clear temporal order (step 1 → step 2 → step 3)
- User says things like "process", "pipeline", "timeline", "steps", "stages"
- Branches represent cause → effect chains

**Hierarchical signals** — use tree layout when:
- Content has a clear taxonomy (kingdom → phylum → class → order)
- One branch naturally "contains" others (org chart, file system)
- Depth matters more than breadth (3+ levels with few branches)
- User says "hierarchy", "org chart", "breakdown", "taxonomy"

**High-density signals** — use semicircle when:
- 7 or more main branches would make a full circle too crowded
- User says "there are many categories" or the source has 8+ sections

If in doubt, default to **radial**. It works for 90% of content.

## Mode 1: Radial (Default)

Full-circle layout with semantic gravity. Best for 4–6 branches.

```
Center: (cx, cy) = (600, 450)
Branch radius: 260–280px from center
Sub-branch radius: 150–165px from parent
Detail radius: 85–100px from sub-branch
```

### Semantic Gravity

Sort branches so conceptually related topics are adjacent. Add a `group` integer
field to each branch. Same-group branches are placed next to each other.

**Angle calculation:**
1. Sort branches by `group` (ascending)
2. Calculate base step: `(2π - totalGroupGaps) / branchCount`
3. Group gap: `0.12–0.15 radians` extra between groups
4. Starting angle: `-π/2` (top of circle)
5. Apply small random offset per branch: `±0.04–0.08 rad` to avoid mechanical look

```javascript
let angle = -Math.PI / 2;
sortedBranches.forEach((branch, i) => {
  if (i > 0 && isNewGroup(i)) angle += GROUP_GAP;
  const bAngle = angle + (i * 0.05 - 0.08);  // slight jitter
  const bx = cx + Math.cos(bAngle) * BRANCH_RADIUS;
  const by = cy + Math.sin(bAngle) * BRANCH_RADIUS;
  // ... place children in a spread arc centered on bAngle
  angle += baseStep;
});
```

### Child Spread

For each branch, distribute children in an arc centered on the parent angle:

```javascript
const spread = Math.PI * (childCount > 4 ? 0.65 : 0.52);
const startAngle = parentAngle - spread / 2;
children.forEach((child, j) => {
  const childAngle = childCount === 1
    ? parentAngle
    : startAngle + (j / (childCount - 1)) * spread;
  const cx = parentX + Math.cos(childAngle) * SUB_RADIUS;
  const cy = parentY + Math.sin(childAngle) * SUB_RADIUS;
});
```

## Mode 2: Semi-Circular

Half-circle arc for 7+ branches. Prevents the full-circle overcrowding problem.

```
Center: (cx, cy) = (600, 500)  — shifted down to give arc room
Arc: from -π (left) to 0 (right), spanning the top half
Branch radius: 300px (wider than radial to use horizontal space)
Sub-branch radius: 140px from parent
```

### Angle Calculation

Same as radial but constrained to a π arc instead of 2π:

```javascript
const ARC_START = -Math.PI;  // left
const ARC_END = 0;           // right
const arcSpan = ARC_END - ARC_START;  // π
const step = arcSpan / (branchCount - 1);

sortedBranches.forEach((branch, i) => {
  const angle = ARC_START + i * step;
  const bx = cx + Math.cos(angle) * 300;
  const by = cy + Math.sin(angle) * 300;
});
```

**ViewBox:** Use `0 0 1200 1000` (taller) to accommodate the downward-shifted center.

Semantic gravity grouping still applies — sort branches by group before distributing
along the arc. Children spread below/outward from each branch.

## Mode 3: Top-Down Tree

Vertical hierarchy. Best for taxonomies, org charts, and deeply nested content.

```
Root: centered at (600, 60)
Level spacing: 140px vertical gap between levels
Sibling spacing: computed to avoid overlap (see algorithm below)
```

### Algorithm

Use a two-pass approach:

**Pass 1 — Bottom-up width computation:**
Each leaf node has a width of `pillWidth + 20px gap`. Each parent's width is the
sum of its children's widths. This guarantees no overlap.

```javascript
function computeWidth(node) {
  if (!node.children.length) return pillWidth(node) + 20;
  node.subtreeWidth = node.children.reduce((sum, c) => sum + computeWidth(c), 0);
  return node.subtreeWidth;
}
```

**Pass 2 — Top-down positioning:**
Place each node at the horizontal center of its allocated width.

```javascript
function positionNode(node, left, top) {
  node.x = left + node.subtreeWidth / 2;
  node.y = top;
  let childLeft = left;
  node.children.forEach(child => {
    positionNode(child, childLeft, top + LEVEL_GAP);
    childLeft += child.subtreeWidth;
  });
}
```

**Connections:** Use vertical-then-horizontal step connectors instead of Bézier curves:
```
M parentX parentY+h/2  L parentX childY-20  L childX childY-20  L childX childY-h/2
```
This produces clean right-angle paths like a real org chart.

**ViewBox:** Dynamically sized to `(totalWidth + padding) × (levels * LEVEL_GAP + padding)`.

## Mode 4: Left-to-Right Flow

Horizontal chain for processes, pipelines, and cause→effect sequences.

```
Start: (80, centerY)
Step spacing: 200px horizontal gap between stages
Sub-item spacing: 60px vertical gap between items within a stage
```

### Algorithm

Branches represent sequential stages. Each stage is a vertical column of nodes.

```javascript
const STEP_GAP = 200;
const ITEM_GAP = 60;
const startX = 80;
const centerY = viewHeight / 2;

branches.forEach((branch, i) => {
  const stageX = startX + i * STEP_GAP;
  // Branch label centered in its column
  branch.x = stageX;
  branch.y = centerY;

  // Children stacked vertically
  const totalHeight = (branch.children.length - 1) * ITEM_GAP;
  branch.children.forEach((child, j) => {
    child.x = stageX;
    child.y = centerY - totalHeight / 2 + j * ITEM_GAP;
  });
});
```

**Connections between stages:** Horizontal arrows from stage N to stage N+1, drawn as
straight lines with small arrowheads:
```
M stage1.x+hw stage1.y  L stage2.x-hw stage2.y
```

**Connections within stages:** Short vertical lines from branch node to children (same
as radial sub-branch connections but vertical).

**ViewBox:** `(branches.length * STEP_GAP + padding) × (maxChildCount * ITEM_GAP + padding)`.

**Arrow styling:** Add small arrowheads at the end of inter-stage connectors using an
SVG marker definition:
```javascript
<defs>
  <marker id="arrow" viewBox="0 0 10 6" refX="10" refY="3"
    markerWidth="8" markerHeight="6" orient="auto">
    <path d="M 0 0 L 10 3 L 0 6 z" fill="{branchColor}" opacity="0.5"/>
  </marker>
</defs>
```

## Layout Field in Data Structure

Add to `mindmapData`:
```javascript
const mindmapData = {
  central: "Topic Name",
  layout: "radial",  // "radial" | "semicircle" | "tree" | "flow" | omit for auto
  branches: [...]
};
```

## User Override

Users can switch layout after generation:
- "Make this a tree" → set `layout: "tree"`, recompute positions, regenerate artifact
- "Switch to flow layout" → set `layout: "flow"`, recompute, regenerate
- "Go back to radial" → set `layout: "radial"`, recompute, regenerate

This is a conversational edit — follow the same pattern (update data, keep engine, regenerate).
