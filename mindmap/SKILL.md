---
name: mindmap
description: >
  CRITICAL — if the user asks for a mind map, concept map, knowledge map, visual summary,
  or says "map this out", "map this", "mind map this", "visualize the structure", "break this
  down visually", or "turn this into a mind map", you MUST use this skill. Stop everything
  and read this SKILL.md immediately. The output is ALWAYS a React (.jsx) artifact with an
  interactive radial mind map. Do NOT respond with a plain text summary or outline. Do NOT
  use Mermaid.js. Do NOT use Excalidraw. Always produce a React mind map artifact — no exceptions.
---

# Mind Map Generator

Generate interactive mind maps as React (.jsx) artifacts.

## When You Receive a Request

1. Analyze the input content — extract the central topic and 4–7 main branches
2. Build the `mindmapData` object following the Data Structure section below
3. Generate a React artifact that includes ALL of the Mandatory Code below
4. Optionally read `references/mindmap-best-practices.md` for keyword compression examples

## Content Analysis Strategy

- **Central topic**: One noun phrase, 2–5 words max.
- **Main branches**: 4–7 categories. 5–6 is the sweet spot.
- **Sub-branches**: 2–5 items per branch. Keywords only, not full sentences.
- **Depth limit**: 3 levels max (central → branch → sub-branch).

For long content: identify the main argument → group into thematic clusters → extract
key evidence → discard redundancy. For broad topics: educational overview, foundational
concepts first.

**Branch balance**: Thin branches (1 item) → merge or expand. Fat branches (6+) → split.

### Content Intelligence

**Contradiction detection**: Sources disagree → mark both with ⚡ prefix, add `conflict` field.
**Gap analysis**: Thin evidence → add ❓ prefix, set `type: "inquiry"`.
**Frequency weighting**: Multi-source → `weight` field (1–5).
**Cross-branch links**: Meaningful relationships → `crossLinks` array. Max 3–4.

## Data Structure

```javascript
const mindmapData = {
  central: "Topic Name",
  branches: [
    {
      label: "🎯 Branch Name",  // emoji + 2-5 words
      group: 0,                  // semantic grouping
      children: [
        { label: "Sub-item", detail: "Hover tooltip text" },
        { label: "❓ Gap node", type: "inquiry" },
      ],
    },
  ],
};
```

Fields: `label` (2-5 words), `group` (int, adjacent placement), `detail` (hover text),
`weight` (1-5), `type: "inquiry"`, `conflict` (node ID), `sourceId`, `sources[]`, `crossLinks[]`.

## ═══ MANDATORY CODE — Include ALL of This in Every Artifact ═══

The sections below contain exact code that MUST appear in every generated mind map.
Copy these functions and UI elements verbatim. Do not paraphrase, simplify, or omit them.

### Pill Sizing

```jsx
function getPill(label, depth) {
  var fs = depth === 0 ? 17 : depth === 1 ? 14 : 12;
  var pad = depth === 0 ? 34 : depth === 1 ? 22 : 18;
  return {
    w: Math.max(label.length * fs * 0.62 + pad * 2, 90),
    h: depth === 0 ? 50 : depth === 1 ? 38 : 32,
  };
}
```

### Edge-Anchored Connectors

Lines MUST start and end at the pill EDGE, not the center. Use these two functions:

```jsx
function edgePoint(cx, cy, hw, hh, tx, ty) {
  var dx = tx - cx, dy = ty - cy;
  var dist = Math.sqrt(dx * dx + dy * dy);
  if (dist < 0.01) return { x: cx + hw, y: cy };
  var nx = dx / dist, ny = dy / dist;
  var t = 1.0 / Math.sqrt((nx * nx) / (hw * hw) + (ny * ny) / (hh * hh));
  return { x: cx + nx * t, y: cy + ny * t };
}

function buildCurve(ax, ay, aLabel, aDepth, bx, by, bLabel, bDepth) {
  var pa = getPill(aLabel, aDepth), pb = getPill(bLabel, bDepth);
  var s = edgePoint(ax, ay, pa.w / 2, pa.h / 2, bx, by);
  var e = edgePoint(bx, by, pb.w / 2, pb.h / 2, ax, ay);
  var qx = (s.x + e.x) / 2 + (s.y - e.y) * 0.18;
  var qy = (s.y + e.y) / 2 + (e.x - s.x) * 0.18;
  return "M " + s.x + " " + s.y + " Q " + qx + " " + qy + " " + e.x + " " + e.y;
}
```

When rendering edges, call `buildCurve` with the actual node positions and labels:

```jsx
{edges.map(function(edge, i) {
  var fromN = nodesMap.get(edge.from);
  var toN = nodesMap.get(edge.to);
  if (!fromN || !toN) return null;
  var d = buildCurve(fromN.x, fromN.y, fromN.label, fromN.depth,
                     toN.x, toN.y, toN.label, toN.depth);
  return <path key={"e-" + i} d={d} fill="none" stroke={color}
    strokeWidth={edge.depth === 1 ? 2.5 : 1.6}
    strokeOpacity={edge.depth === 1 ? 0.45 : 0.25} strokeLinecap="round" />;
})}
```

NEVER write `M ${a.x} ${a.y} Q ... ${b.x} ${b.y}` — that draws center-to-center.

### Node Rendering — Fully Opaque Pills

Pill `<rect>` elements MUST be 100% opaque so edges do not bleed through.
CRITICAL: Do NOT use `opacity`, `fillOpacity`, `rgba()`, or 8-digit hex codes to lighten
colors. If you need a lighter color, calculate a solid hex mixed with white. Force `fillOpacity={1}`.

Copy this pattern exactly:

```jsx
{nodes.map(function(node) {
  var pill = getPill(node.label, node.depth);
  var w = pill.w, h = pill.h;
  // Use SOLID colors only. No rgba, no opacity.
  return (
    <g key={node.id} transform={"translate(" + node.x + "," + node.y + ")"}>
      {/* Pill — Forced 100% opacity */}
      <rect x={-w/2} y={-h/2} width={w} height={h}
        rx={node.depth === 0 ? 13 : 19} fill={bgColor} fillOpacity={1} />
      <text textAnchor="middle" dy="0.35em" fontSize={fontSize}
        fontWeight={fontWeight} fill={textColor}
        style={{ fontFamily: "system-ui, sans-serif", pointerEvents: "none" }}>
        {node.label}
      </text>
    </g>
  );
})}
```

### 💾 Save Button & Component Structure

Every mind map MUST include a Save button. To ensure this is not skipped, you MUST use
this exact structural skeleton for your main component:

```jsx
export default function MindMap() {
  var [saveMsg, setSaveMsg] = useState(null);
  var [palName, setPalName] = useState("Bauhaus");

  // 1. Paste handleSave function here exactly as written:
  async function handleSave() {
    var slug = mindmapData.central.toLowerCase().replace(/[^a-z0-9]+/g, "-").replace(/(^-|-$)/g, "");
    var key = "maps:" + slug;
    var existing = null;
    try { existing = await window.storage.get(key); } catch(e) {}
    var now = new Date().toISOString();
    var record = {
      data: mindmapData, palette: palName, layout: "radial",
      createdAt: existing ? JSON.parse(existing.value).createdAt : now,
      updatedAt: now, tags: [],
    };
    try {
      await window.storage.set(key, JSON.stringify(record));
      setSaveMsg(existing ? "Updated ✓" : "Saved ✓");
    } catch(e) { setSaveMsg("Error"); }
    setTimeout(function() { setSaveMsg(null); }, 2000);
  }

  return (
    <div style={{ position: "relative", width: "100%", height: "100vh" }}>
      {/* 2. Toolbar MUST contain the Save Button */}
      <div style={{ position: "absolute", top: 16, left: 16, display: "flex", gap: 8, zIndex: 10 }}>
        {/* Palette Picker goes here */}
        
        {/* MANDATORY SAVE BUTTON */}
        <button onClick={handleSave} style={{
          height: 32, padding: "0 16px", border: "1px solid #ddd", borderRadius: 7,
          background: saveMsg ? "#e8f5ee" : "#fff", cursor: "pointer",
          fontSize: 13, fontWeight: 700, fontFamily: "system-ui, sans-serif",
          color: saveMsg ? "#2D6A4F" : "#555",
        }}>
          {saveMsg || "💾 Save"}
        </button>
      </div>

      {/* 3. SVG Canvas goes here */}
    </div>
  );
}
```

### Palette System

Include all four palettes and a runtime switcher:

```jsx
var PALETTES = {
  Bauhaus: ["#D64045","#1D3557","#E9B44C","#2D6A4F","#7B2D8E","#E07A5F","#457B9D"],
  "Ocean Sunset": ["#E76F51","#2A9D8F","#E9C46A","#264653","#F4A261","#287271","#BC4749"],
  "Nordic Forest": ["#5B8C5A","#4A6FA5","#C17C74","#7B6D8D","#D4A574","#4D8B8B","#8B6F47"],
  "Pastel Garden": ["#7EB8DA","#B5C99A","#E8A87C","#9B8EC1","#F2CC8F","#81B29A","#E07A5F"],
};
```

Add a 🎨 button that shows a dropdown to switch palettes at runtime.

### Layout Algorithm

Radial layout with semantic gravity. Branches sorted by `group`, distributed around
a circle with gaps between groups:

```jsx
function buildLayout(data) {
  var nodes = [], edges = [], cx = 600, cy = 450;
  nodes.push({ id: "root", label: data.central, x: cx, y: cy, depth: 0,
    childIds: [], collapsed: false, bi: -1, detail: null });
  var branches = data.branches || [];
  var sorted = branches.map(function(b, i) { return Object.assign({}, b, { oi: i }); });
  sorted.sort(function(a, b) { return (a.group || 0) - (b.group || 0); });
  var groups = [], lastG = null;
  sorted.forEach(function(b, i) { if (b.group !== lastG) { groups.push(i); lastG = b.group; } });
  var gapExtra = 0.14, totalGap = Math.max(0, groups.length - 1) * gapExtra;
  var baseStep = (2 * Math.PI - totalGap) / sorted.length;
  var angle = -Math.PI / 2;
  sorted.forEach(function(branch, i) {
    if (i > 0 && groups.indexOf(i) !== -1) angle += gapExtra;
    var bAngle = angle + (i * 0.05 - 0.08);
    var bx = cx + Math.cos(bAngle) * 260, by = cy + Math.sin(bAngle) * 260;
    var bId = "b-" + branch.oi;
    nodes.push({ id: bId, label: branch.label, x: bx, y: by, depth: 1,
      childIds: [], collapsed: false, bi: branch.oi, detail: branch.detail || null });
    nodes[0].childIds.push(bId);
    edges.push({ from: "root", to: bId, bi: branch.oi, depth: 1 });
    var kids = branch.children || [];
    kids.forEach(function(child, j) {
      var cc = kids.length, spread = Math.PI * (cc > 4 ? 0.65 : 0.5);
      var sa = bAngle - spread / 2;
      var ca = cc === 1 ? bAngle : sa + (j / (cc - 1)) * spread;
      var sx = bx + Math.cos(ca) * 150, sy = by + Math.sin(ca) * 150;
      var cId = "s-" + branch.oi + "-" + j;
      nodes.push({ id: cId, label: child.label, x: sx, y: sy, depth: 2,
        childIds: [], collapsed: false, bi: branch.oi, detail: child.detail || null });
      nodes.find(function(n) { return n.id === bId; }).childIds.push(cId);
      edges.push({ from: bId, to: cId, bi: branch.oi, depth: 2 });
    });
    angle += baseStep;
  });
  return { nodes: nodes, edges: edges };
}
```

### Required Interaction Features

- **Pan**: mousedown drag on canvas background
- **Zoom**: mouse wheel, plus +/− buttons in toolbar
- **Collapse/expand**: click branch nodes to toggle children
- **Hover tooltip**: fixed position (top-left), shows `detail` text if present
- **Fullscreen**: ⛶ button, fills viewport, Escape to exit

## ═══ END MANDATORY CODE ═══

## Conversational Editing

After first generation, users refine through conversation. Only change `mindmapData`.
Do NOT re-read references. Do NOT regenerate engine code.

| User says | Operation |
|---|---|
| "Add X under Y" | Add child node |
| "Remove X" | Delete node |
| "Move X to Y" | Remove from old parent, add to new |
| "Rename X to Y" | Update label |
| "Merge X and Y" | Combine children |
| "Expand X" | Add 2–4 sub-items |
| "Simplify" | Prune depth-3, merge thin branches |
| "Change palette" | Change default palette |
| "Save this map" | Already handled by 💾 button |
| "Show my atlas" | Generate atlas per `references/atlas-storage.md` |

Edit response: State change (1 sentence) → regenerate → stop.

## Edge Cases

- **Very short input**: Brainstorm-style map exploring facets.
- **Very long input**: Compress to 4–7 branches. Mention what was cut.
- **Ambiguous**: Generate first, ask second.
- **Non-English**: Same language as input.

## What NOT To Do

- Do NOT generate connectors using center coordinates — use `edgePoint()` + `buildCurve()`
- Do NOT omit the 💾 Save button — use the component skeleton above with `handleSave`
- Do NOT use `opacity`, `rgba()`, or alpha channels to lighten pill colors — use solid hex only, force `fillOpacity={1}`
- Do NOT respond with a plain text summary — always produce a React artifact
- Do NOT use Mermaid.js or Excalidraw
- Do NOT skip the palette switcher

## Output Pattern

**First generation:**
1. Brief explanation of extracted structure (1–2 sentences)
2. React artifact with mindmapData + all mandatory code above
3. Offer to adjust

**Edits:**
1. State the change (1 sentence)
2. Regenerate with updated data only
3. Stop.
