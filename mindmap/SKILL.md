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

Generate interactive mind maps rendered as React (.jsx) artifacts. The maps are visually rich,
zoomable, and follow established mind map best practices for clarity and recall.

## When You Receive a Request

1. **First mind map in a conversation**: Read `references/mindmap-best-practices.md` for
   design principles and `references/export-patterns.md` for export code patterns.
2. **Follow-up requests in the same conversation** (e.g., "expand this branch", "switch
   palette", "export as PNG"): Skip reading the reference files — they're already in
   context. Go straight to the change.
3. **Analyze the input content** — extract the core topic and 4–7 main branches
4. **Generate the React artifact** following the template pattern below

## Content Analysis Strategy

Before building the map, think carefully about structure:

- **Central topic**: One clear noun phrase (2–5 words max). This is the heart of the map.
- **Main branches**: 4–7 primary categories. These are the "chapters" of the content.
  Too few = not useful. Too many = overwhelming. 5–6 is the sweet spot.
- **Sub-branches**: 2–5 items per main branch. Use keywords, not full sentences.
  See the Keyword Compression section in `references/mindmap-best-practices.md` for
  20 before/after examples of how to compress well.
- **Depth limit**: Go no deeper than 3 levels (central → branch → sub-branch → detail).
  Deeper trees lose the visual advantage of a mind map.

When analyzing long or complex content (articles, documents, book chapters):
- First identify the author's main argument or thesis → this becomes the central topic
- Group supporting points into thematic clusters → these become main branches
- Extract key evidence, examples, or details → these become sub-branches
- Discard redundancy ruthlessly — mind maps are about compression, not completeness

When the input is a broad topic (e.g., "machine learning") rather than a specific document:
- Use your knowledge to create a well-structured educational overview
- Prioritize the most important/foundational concepts
- Arrange branches in a logical learning order (foundational → advanced, clockwise)

### Branch Balance Analysis

After extracting your initial structure, check for imbalance before generating the artifact:

- **Thin branches** (only 1 sub-item): This usually means the branch is either too narrow
  (merge it into a sibling) or under-explored (you missed supporting content). If the input
  is a document, re-read it for material you may have compressed too aggressively. If it's
  a broad topic, use your knowledge to flesh it out.
- **Fat branches** (6+ sub-items): The branch is probably two concepts jammed together.
  Split it. Look for a natural fault line — there almost always is one.
- **Orphan details** (a depth-3 node with no siblings): Promote it to depth-2, or ask
  the user whether this detail deserves its own branch.
- **When you can't fix imbalance from the source material**: Tell the user explicitly.
  Say something like: "The source covers [Branch X] in much more depth than [Branch Y].
  Want me to research [Branch Y] further, or keep the map as-is?" Generate the map
  anyway so they have something to react to — don't block on the question.

### Content Intelligence

These steps turn a mind map from a passive summary into an analytical tool. Run them
after extracting branches but before generating the artifact.

**Contradiction detection.** Scan your extracted nodes for claims that conflict with each
other — this is common when synthesizing multiple sources. When you find a contradiction,
don't silently pick a side. Instead, mark both nodes with a ⚡ prefix and add a `conflict`
field to the data structure linking the two node IDs. The renderer will draw a red dashed
line between conflicting nodes so the user can see the tension at a glance. In your
response text, briefly explain the contradiction.

**Gap analysis (Inquiry Nodes).** If a branch is structurally important but the source
material is thin, vague, or missing on that topic, add a node with a ❓ prefix and set
its `type` field to `"inquiry"`. These render with a dashed border to signal they're
questions, not assertions. They serve two purposes: they honestly flag where the map is
speculative, and they give the user a natural prompt to ask for more depth. Only add
inquiry nodes when the gap is genuinely significant — not as decoration.

**Frequency-based weighting.** When processing multi-source input, track how many
distinct sources mention each concept. Store this as a `weight` field (1–5 scale) on
each node. Weight affects rendering: heavier nodes get slightly larger pills and bolder
text. This helps users see which ideas have broad consensus versus which appear in only
one source. For single-source or knowledge-based maps, omit the weight field entirely —
don't manufacture fake emphasis.

**Cross-branch links (synaptic mapping).** Some ideas genuinely belong in one branch but
have a meaningful relationship to a node in another branch. When you detect these
interdependencies, add them to a top-level `crossLinks` array in the data structure.
Each cross-link specifies two node IDs and a brief label describing the relationship.
These render as thin dotted curved lines in a neutral gray, with a small label at the
midpoint. Use sparingly — more than 3–4 cross-links clutters the map. Only add them
when the relationship is non-obvious and adds real insight.

## Visual Design Specifications

### Layout

Read `references/layout-engine.md` for the full layout algorithms. The skill supports
four layout modes — the engine auto-selects based on content shape, but the user can
override. Quick summary:

| Mode | When auto-selected | Shape |
|------|-------------------|-------|
| **Radial** (default) | 4–6 branches | Full circle, semantic gravity groups |
| **Semi-circular** | 7+ branches | Half-circle arc, avoids overcrowding |
| **Top-down tree** | Sequential content (processes, timelines) | Vertical hierarchy |
| **Left-to-right flow** | Cause→effect, pipelines, workflows | Horizontal chain |

Add a `layout` field to `mindmapData` to force a mode: `"radial"`, `"semicircle"`,
`"tree"`, or `"flow"`. Omit to auto-detect. The `group` field still works for semantic
gravity in radial and semi-circular modes.

### Color System

Each main branch gets its own color from a named palette. The component ships with all
four palettes defined in `references/mindmap-best-practices.md` and lets the user switch
between them at runtime via a palette selector in the control bar.

Store the palettes as a constant object:

```javascript
const PALETTES = {
  "Ocean Sunset": ["#E76F51", "#2A9D8F", "#E9C46A", "#264653", "#F4A261", "#287271", "#BC4749"],
  "Nordic Forest": ["#5B8C5A", "#4A6FA5", "#C17C74", "#7B6D8D", "#D4A574", "#4D8B8B", "#8B6F47"],
  "Bauhaus":       ["#D64045", "#1D3557", "#E9B44C", "#2D6A4F", "#7B2D8E", "#E07A5F", "#457B9D"],
  "Pastel Garden": ["#7EB8DA", "#B5C99A", "#E8A87C", "#9B8EC1", "#F2CC8F", "#81B29A", "#E07A5F"],
};
```

Choose the default palette based on the content's tone — professional/technical → Bauhaus
or Nordic, creative/personal → Ocean Sunset or Pastel Garden. The user can override this
at any time.

Within each palette:
- Branch colors: used directly for depth-1 nodes
- Sub-branch colors: same hue, lightened (+30% via a `lighten()` helper)
- Central node: always a neutral dark (`#0d1b2a` or `#1a1a2e`) — doesn't change with palette
- Background: transparent (inherits the chat theme)

When the user switches palette, all branch colors, sub-branch colors, edge colors,
and cross-link label colors update reactively. The central node stays fixed.

### Typography

- Central topic: bold, 16–18px
- Main branches: semi-bold, 13–15px
- Sub-branches: regular weight, 11–13px
- Detail nodes: regular, 10–12px
- Use the system font stack — it renders well across platforms

### Connections

- Use curved paths (quadratic Bézier curves), not straight lines
- Line thickness decreases with depth: 3px → 2px → 1.5px → 1px
- Lines inherit the color of their parent branch (with slight opacity reduction at depth)

**Edge anchoring (critical):** Lines must connect from the pill edge of the parent to the
pill edge of the child — never center-to-center, as that causes lines to overlap with text.
Compute the anchor point by intersecting a ray from the parent center toward the child
center with the parent's pill boundary. For a pill of half-width `hw` and half-height `hh`:

```javascript
function edgePoint(cx, cy, hw, hh, targetX, targetY) {
  const dx = targetX - cx, dy = targetY - cy;
  const angle = Math.atan2(dy, dx);
  // Pill = rectangle with semicircle caps; approximate with ellipse intersection
  const ex = hw * Math.cos(angle);
  const ey = hh * Math.sin(angle);
  const scale = Math.min(hw / (Math.abs(ex) || 1), hh / (Math.abs(ey) || 1));
  return { x: cx + ex * scale, y: cy + ey * scale };
}
```

Use `edgePoint` for both the start (parent → child direction) and end (child → parent
direction) of each curve. The complete curve function:

```javascript
function curve(fromId, toId, nodesMap, pillFn) {
  const a = nodesMap.get(fromId), b = nodesMap.get(toId);
  if (!a || !b) return "";
  const pa = pillFn(a.label, a.depth), pb = pillFn(b.label, b.depth);
  const s = edgePoint(a.x, a.y, pa.w/2, pa.h/2, b.x, b.y);
  const e = edgePoint(b.x, b.y, pb.w/2, pb.h/2, a.x, a.y);
  const qx = (s.x+e.x)/2 + (s.y-e.y)*0.15;
  const qy = (s.y+e.y)/2 + (e.x-s.x)*0.15;
  return `M ${s.x} ${s.y} Q ${qx} ${qy} ${e.x} ${e.y}`;
}
```

NEVER use `M ${a.x} ${a.y} ... ${b.x} ${b.y}` — that draws center-to-center
and causes lines to overlap with text labels inside the pills.

### Node Styling

- Central node: rounded rectangle with subtle shadow, filled background
- Branch nodes: pill-shaped (border-radius: 20px), with colored background
- Sub-branch nodes: smaller pills, lighter background
- Detail nodes: just text with a subtle left border (no background)
- Add a relevant emoji prefix to each main branch for quick visual scanning

**Text containment (critical):** Text must never extend beyond its pill. Compute pill
width from actual text, not estimates. Use a temporary SVG `<text>` element with
`getBBox().width` to measure accurately, then add padding (28px/side for depth-0,
20px for depth-1, 16px for depth-2). If measuring before SVG mount, use a conservative
character-width estimate of `fontSize * 0.62` per character (not 0.55 — too tight for
wide glyphs and emojis), plus the same per-side padding.

**Weighted nodes** (`weight` field present): Scale the pill width by `1 + (weight - 1) * 0.06`
and bump font-weight up one step (400→500, 500→600). This creates a subtle visual hierarchy
where high-frequency concepts feel slightly more prominent without breaking the layout.

**Inquiry nodes** (`type: "inquiry"`): Render with a dashed border (`strokeDasharray: "4 3"`)
instead of a solid fill. Use a slightly muted version of the branch color. The dashed border
signals "this is a question, not a statement" without needing a legend.

**Conflict nodes** (`conflict` field present): Both conflicting nodes render normally, but
the SVG includes a red dashed curved line (`stroke: "#D64045"`, `strokeDasharray: "6 4"`,
`opacity: 0.5`) connecting the two node IDs. Draw this line in a separate layer above the
normal edges but below the nodes so it doesn't obscure labels.

### Cross-Branch Links

Cross-links render as thin dotted curves (`stroke: "#999"`, `strokeDasharray: "3 3"`,
`strokeWidth: 1`, `opacity: 0.4`) connecting two nodes that live in different branches.
At the midpoint of the curve, place a small text label (9px, gray) describing the relationship.
Use a gentle arc that routes outside the main radial structure to avoid visual confusion with
branch edges. The cross-link layer renders above normal edges but below nodes.

### Canvas

- Default viewBox: `0 0 1200 900` (landscape, gives breathing room)
- Central node at `(600, 450)`
- Main branch radius: 250–300px from center
- Sub-branch radius: 130–170px from parent
- Include pan and zoom controls (mouse wheel + drag, plus +/- buttons)
- Include a "fit to view" reset button

## React Component Structure

Build a single self-contained `.jsx` file with:

1. **Data structure** — a nested object representing the mind map tree:
```javascript
const mindmapData = {
  central: "Topic Name",
  layout: "radial",                        // optional: "radial"|"semicircle"|"tree"|"flow"|omit for auto
  sources: [                               // optional: reference sources
    { id: "s1", label: "Newport 2016", url: "https://..." },
    { id: "s2", label: "Wikipedia", url: "https://..." }
  ],
  crossLinks: [                            // optional: inter-branch relationships
    { from: "s-0-2", to: "s-3-1", label: "enables" }
  ],
  branches: [
    {
      label: "🎯 Branch Name",
      sourceId: "s1",                      // optional: links node to a source
      children: [
        {
          label: "Sub-item 1",
          detail: "Expanded context shown on hover",  // optional: tooltip text
          weight: 3,                       // optional: 1–5, frequency across sources
          sourceId: "s2",
          children: [...]
        },
        { label: "❓ Under-explored area", type: "inquiry" },   // gap node
        { label: "⚡ Claim A conflicts", conflict: "s-2-1" }   // contradiction
      ]
    }
  ]
};
```

**Field reference:**
- `detail` (string, optional): A 1–2 sentence expansion of the keyword label. Shown in
  the fixed tooltip on hover, and inline beneath the node in reading mode. When rendered
  inline, always truncate at word boundaries — never cut mid-word. If the text exceeds
  ~80 characters, find the last space before the limit and append "…". Only add detail
  when the compressed label genuinely loses important nuance.
- `weight` (number 1–5, optional): How many sources mention this concept. Affects pill
  size and font boldness. Omit for single-source or knowledge-based maps.
- `type: "inquiry"` (optional): Marks a gap node. Renders with dashed border and ❓ prefix.
- `conflict` (node ID string, optional): Points to the contradicting node. Both nodes
  should have ⚡ prefixes. Renderer draws a red dashed line between them.
- `sourceId` (string, optional): Links to a source in the `sources` array.
- `crossLinks` (array, optional): Top-level inter-branch relationships. Max 3–4.
- `layout` (string, optional): `"radial"`, `"semicircle"`, `"tree"`, `"flow"`. Omit for auto-detection. See `references/layout-engine.md`.

Each branch also accepts:
- `group` (integer, optional): Semantic grouping number. Branches with the same group are
  placed adjacently on the circle. Use when the content has natural clusters (e.g., group 0
  for "learning methods", group 1 for "obstacles", group 2 for "execution strategies").

When the input content comes from identifiable sources (articles, papers, books, URLs),
populate the `sources` array and tag nodes with `sourceId` to trace where each idea came from.
Nodes with a source get a small link icon (🔗) on hover that opens the URL in a new tab.
The map footer should show a numbered source list when any sources exist. If the input is a
broad topic generated from your own knowledge, omit the sources array entirely — don't
fabricate references.

2. **Layout engine** — computes (x, y) positions using the auto-selected or user-specified layout mode. See `references/layout-engine.md` for all four algorithms.

3. **SVG renderer** — draws connections (curved paths) and nodes (groups of rect + text). Uses the color system above.

4. **Interaction layer**:
   - Pan: click-and-drag on the canvas background
   - Zoom: mouse wheel or +/- buttons
   - Collapse/expand: click a branch node to toggle its children
   - Hover: subtle scale-up (1.05x) and shadow on nodes
   - **Hover tooltip**: If a node has a `detail` field, show a floating tooltip card
     positioned at a FIXED location (e.g., top-left of the canvas at `position: absolute;
     top: 12px; left: 12px`), not near the node itself. This prevents the tooltip from
     overlapping the map or flickering as the mouse moves. Style it as a card with subtle
     shadow, max-width 280px, font-size 12px, good line-height (1.5). Show the node label
     as a colored header, then the detail text as body copy, then the source if present.
   - **Progressive disclosure toggle**: A "Details" button in the control bar toggles
     between two modes: (a) *Map mode* (default) — keyword labels only, detail shown in
     the fixed tooltip on hover; (b) *Reading mode* — detail text rendered beneath each
     node label. In reading mode, render detail text using `foreignObject` with these rules:
     - **Truncate at word boundaries**, never mid-word. Use a helper like:
       `text.split(' ').reduce((acc, w) => (acc + ' ' + w).length <= limit ? acc + ' ' + w : acc, '').trim() + '…'`
     - Max ~12 words or ~80 characters, whichever is shorter
     - Style: 9px font, muted color (#888), `line-height: 1.3`, `text-align: center`
     - Give the foreignObject a `width` matching the pill width and a fixed `height: 40px`
     - Position it `y = h/2 + 6` (a clear gap below the pill, not flush against it)
     When reading mode is active, the layout should expand the vertical spacing between
     sub-branches by ~30% to make room for the detail text.
   - **Focus mode (branch isolation)**: Each main branch node gets a small ◎ icon that
     appears on hover. Clicking it enters focus mode: all other branches fade to 8% opacity,
     the focused branch and its children smoothly animate to center, and the view resets to
     fit just that branch. A "Back to full map" button appears in the control bar. This makes
     dense maps navigable without needing separate pages. In focus mode, cross-links that
     connect to the focused branch remain visible; others fade with the rest.
   - **Palette selector**: A small dropdown or row of color swatches in the control bar
     showing the 4 named palettes. Each swatch is a row of tiny circles (6–8px) previewing
     the palette's colors. Clicking a swatch applies it instantly — all node and edge colors
     update reactively via React state. The selector sits after the Details toggle and before
     the export buttons. Keep it compact: a single 🎨 button that expands a popover on
     click works well to avoid toolbar clutter.
   - **Fullscreen toggle**: A ⛶ button at the end of the control bar. Clicking it expands
     the mind map to fill the entire viewport (`position: fixed`, `100vw × 100vh`,
     `z-index: 10000`). The button label swaps to ✕ while fullscreen. Three exit methods:
     clicking ✕, pressing Escape, or clicking ⛶ again. Lock `body` overflow while
     fullscreen to prevent background scrolling. The canvas area should flex to fill
     all available height below the toolbar.
   - Fit-to-view button resets transform

5. **Legend/controls** — small control bar at top-right with zoom buttons and fit-to-view

6. **Export toolbar** — a row of export buttons (SVG, PNG, PDF) placed alongside the zoom controls.
   All export logic is self-contained — no external libraries required.

## Export Implementation

Read `references/export-patterns.md` for complete code patterns. Summary of what each does:

- **SVG**: Serializes a cleaned SVG clone with portable font stack and white background
- **PNG**: Renders SVG onto offscreen canvas at 2× resolution for retina output
- **PDF**: Opens browser print dialog with landscape layout for true vector output
- **Mermaid**: Generates `.mermaid` text compatible with Obsidian, GitHub, mermaid.live
- **Markdown**: Generates `.md` outline with heading hierarchy and source footnotes
- **Embed HTML**: Generates a single self-contained `.html` file with a vanilla JS mind
  map engine, the full data structure, and all interactivity (pan, zoom, collapse, palette
  switching, tooltips). The file has zero external dependencies and can be hosted anywhere
  (GitHub Pages, S3, Netlify, personal site) or embedded via `<iframe>`. On click, it
  also copies an iframe snippet to the clipboard and flashes confirmation.

**Critical font rule for exports**: The in-browser rendering uses `system-ui` which always
resolves. But exported SVGs must use `Arial, Helvetica, 'Segoe UI', Roboto, sans-serif` —
system-ui and -apple-system resolve to nothing on Android, Windows PDF viewers, and most
non-browser contexts. This is handled in the `prepareSvgForExport` function.

**Export button UI**: Three groups separated by dividers — Visual: SVG, PNG, PDF;
Interchange: MD, MMD; Embed: `</>`. Plus a 💾 save button (persists map to storage for
the atlas). Flash ✓ for 1.5s on click. See `references/atlas-storage.md` for persistence,
atlas viewer, and auto-linking. Filename = slugified central topic.

**Expand before export**: Always export ALL nodes including collapsed ones. The export
functions receive full `layout.nodes`, not the filtered visible subset.

## Quality Checklist

Quick final scan — only the items easy to miss. If the prose instructions above were
followed, most quality issues are already handled.

- [ ] No node label exceeds 5 words (the most common violation — compress harder)
- [ ] 4–7 main branches, each with 2–5 sub-items (count them)
- [ ] Every main branch has an emoji prefix
- [ ] Connections anchor at pill edges — no line overlaps any text
- [ ] All text labels fit within their pill with visible padding on both sides
- [ ] No text overlaps at default zoom (zoom out and check)
- [ ] Reading mode detail text truncates at word boundaries, never mid-word
- [ ] Contradictions and inquiry nodes are genuine, not decorative
- [ ] Cross-links limited to 3–4 max
- [ ] The map tells a coherent story you could present to someone

## Edge Cases

- **Very short input** (a word or sentence): Brainstorm-style map exploring facets of the topic.
- **Very long input** (5000+ words): Compress to 4–7 branches, 3–4 items each. Mention what was cut.
- **Ambiguous input**: Generate first with your best interpretation, ask second.
- **Lopsided source**: Flag imbalance after generating, offer to research thin branches.
- **Non-English input**: Generate the map in the same language as the input.

## Conversational Editing

After the first mind map is generated, users will often want to refine it through
conversation. This is the most common interaction pattern — and the most important to
get right. The goal: make edits feel instant and cheap, not like starting over.

### Core Principle

The `mindmapData` object is the single source of truth. On edit requests, change the
data object and regenerate the artifact. The entire rendering engine, export functions,
interaction layer, and UI code stay identical — only the data block at the top of the
file changes. This means the "expensive" part (the engine) is copy-pasted unchanged,
and only the "cheap" part (the data) is rewritten.

Do NOT re-analyze the original source content. Do NOT re-read the reference files.
The map's current structure is the starting point — apply the requested change to it.

### Recognizing Edit Intent

Map natural language to one of these operations:

| User says (examples) | Operation | What changes in data |
|---|---|---|
| "Add X under Y" / "include X" | **Add node** | New child in target branch's `children` array |
| "Remove X" / "delete the Z branch" | **Remove node** | Delete from `children` + remove any cross-links referencing it |
| "Move X to Y" / "X belongs under Y" | **Move node** | Remove from old parent, add to new parent |
| "Rename X to Y" / "change the label" | **Relabel** | Update `label` field |
| "Merge X and Y" | **Merge branches** | Combine children into one branch, pick the better label |
| "Split X into two" | **Split branch** | Create two branches, redistribute children logically |
| "Expand X" / "more detail on X" | **Deepen branch** | Add 2–4 sub-items using your knowledge or the original source |
| "Simplify" / "too much detail" | **Prune** | Remove depth-3 nodes, merge thin branches |
| "Link X to Y" / "these are related" | **Add cross-link** | Add entry to `crossLinks` array |
| "Change palette to Z" | **Palette swap** | Change the default palette name constant |
| "Change the style" / "make it darker" | **Visual tweak** | Adjust central node color, palette, or layout constants |
| "Add source for X" | **Attribution** | Add to `sources` array + set `sourceId` on the node |
| "Make it a tree" / "use flow layout" | **Layout switch** | Set `layout` field + recompute positions per `references/layout-engine.md` |
| "Save this map" / "show my atlas" | **Storage** | Save/load/list maps per `references/atlas-storage.md` |

### Edit Response Pattern

When handling an edit request:

1. **State the change** in one sentence: "I've moved 'Social Media' from Enemies to Training
   and added it as a sub-branch of 'Embrace boredom'."
2. **Regenerate the artifact** with the updated `mindmapData` object. Copy the entire
   rendering engine unchanged — only the data block at the top differs.
3. **Don't re-explain the map**. The user already knows what it contains. Don't list all
   branches, don't re-describe features, don't re-offer to adjust.

### Handling Ambiguous Edits

If the user's request is unclear (e.g., "fix the structure"), make your best interpretation
and execute it, then briefly explain what you changed and why. Don't ask for clarification
before acting — show them the result and let them redirect if needed.

If an edit would break a structural rule (e.g., merging would leave a branch with 7+
items), do the edit but flag the violation: "I merged those, but the branch now has 8
sub-items — want me to split it?"

### Multi-Edit Turns

Users sometimes request several changes at once: "Move X to Y, rename Z, and add a
branch for W." Handle all changes in a single artifact regeneration — don't generate
intermediate versions. List each change briefly in your response.

## What NOT To Do
- Do NOT respond with a plain text summary or bullet-point outline
- Do NOT use Mermaid.js, Excalidraw, or any other diagram tool
- Do NOT generate SVG directly — always wrap in a React component
- Do NOT draw connectors from node centers — always use `edgePoint()` for edge anchoring
- Do NOT omit the 💾 Save button — it must appear in the toolbar of every generated map
- Do NOT skip the palette selector — all 4 palettes must be switchable at runtime
- Do NOT use `opacity < 1` on pill fills — pills must be fully opaque so edges behind are hidden
- Do NOT put explanatory prose inside the artifact — text goes in your chat response

## Example Output Pattern

When responding to the user:

**First generation:**
1. Briefly explain what structure you extracted (1–2 sentences)
2. Generate the React artifact
3. Offer to adjust: "Want me to expand any branch, change the structure, or adjust the visual style?"

**Subsequent edits:**
1. State what you changed (1 sentence)
2. Regenerate the artifact with updated data
3. Stop. Don't re-describe the map.
