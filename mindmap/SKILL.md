---
name: mindmap
description: >
  Generate interactive, visually polished mind maps from any content — text, documents, conversations,
  topics, brainstorms, outlines, or ideas. Use this skill whenever the user asks for a mind map,
  concept map, topic map, visual summary, visual outline, brainstorm visualization, or knowledge map.
  Also trigger when the user says things like "map this out", "visualize the structure of",
  "show me the relationships between", "break this down visually", "give me an overview diagram of",
  or "turn this into a mind map". Works with any input: pasted text, uploaded files, URLs, topics,
  or freeform ideas. If the user wants a visual, hierarchical breakdown of any subject — use this skill.
---

# Mind Map Generator

Generate interactive mind maps rendered as React (.jsx) artifacts. The maps are visually rich,
zoomable, and follow established mind map best practices for clarity and recall.

## When You Receive a Request

1. **Read the reference** at `references/mindmap-best-practices.md` for design principles
2. **Analyze the input content** — extract the core topic and 4–7 main branches
3. **Generate the React artifact** following the template pattern below

## Content Analysis Strategy

Before building the map, think carefully about structure:

- **Central topic**: One clear noun phrase (2–5 words max). This is the heart of the map.
- **Main branches**: 4–7 primary categories. These are the "chapters" of the content.
  Too few = not useful. Too many = overwhelming. 5–6 is the sweet spot.
- **Sub-branches**: 2–5 items per main branch. Use keywords, not full sentences.
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

### Layout: Semantic Radial Tree

Nodes radiate outward from a central topic. Main branches are distributed around the center,
but their angular positions are not purely mathematical — they encode meaning.

**Semantic gravity**: Before assigning angles, sort branches so that conceptually related
topics are adjacent on the circle. Think of the circle as a clock face where proximity
implies relatedness. For example, in a "Machine Learning" map, "Supervised Learning" and
"Unsupervised Learning" should sit next to each other, while "Hardware Requirements" belongs
on the opposite side. In a business strategy map, "Revenue" and "Costs" are neighbors;
"Company Culture" sits across from them.

To implement this, add a `group` integer field to each branch in the data structure.
Branches with the same group number are placed adjacently. Within a group, order by
conceptual flow (general → specific, cause → effect, or chronological). Between groups,
add a slightly larger angular gap (1.3× the normal spacing) to create visual breathing
room that reinforces the conceptual boundary.

After grouping, calculate angles as:
`baseAngle = (index / totalBranches) * 2π - π/2`
Then apply the group gaps and a small random offset (±8°) to avoid mechanical uniformity.

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
- Lines connect from parent edge to child edge, not center-to-center

### Node Styling

- Central node: rounded rectangle with subtle shadow, filled background
- Branch nodes: pill-shaped (border-radius: 20px), with colored background
- Sub-branch nodes: smaller pills, lighter background
- Detail nodes: just text with a subtle left border (no background)
- Add a relevant emoji prefix to each main branch for quick visual scanning

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

2. **Layout engine** — a function that computes (x, y) positions for every node using the radial algorithm described above. This runs once on mount and recalculates if the data changes.

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

**Critical font rule for exports**: The in-browser rendering uses `system-ui` which always
resolves. But exported SVGs must use `Arial, Helvetica, 'Segoe UI', Roboto, sans-serif` —
system-ui and -apple-system resolve to nothing on Android, Windows PDF viewers, and most
non-browser contexts. This is handled in the `prepareSvgForExport` function.

**Export button UI**: Two groups in the control bar separated by dividers.
Visual exports: SVG, PNG, PDF. Interchange exports: MD, MMD. Flash a ✓ for 1.5s on click.
Filename = slugified central topic.

**Expand before export**: Always export ALL nodes including collapsed ones. The export
functions receive full `layout.nodes`, not the filtered visible subset.

## Quality Checklist

Before finalizing, verify:

- [ ] Central topic is concise (2–5 words)
- [ ] 4–7 main branches, no more
- [ ] Each branch has 2–5 sub-items
- [ ] No node label exceeds 5 words (compress ruthlessly)
- [ ] Every main branch has an emoji prefix
- [ ] Colors are distinct and harmonious
- [ ] Curved connections, not straight lines
- [ ] Pan/zoom works
- [ ] Collapse/expand works on branch nodes
- [ ] No text overlaps at default zoom
- [ ] Export buttons (SVG, PNG, PDF, MD, Mermaid) are present and functional
- [ ] Export captures full map regardless of collapse state
- [ ] Exported SVG renders correctly when opened standalone (no missing fonts or CSS vars)
- [ ] Exported PNG is crisp at 2× resolution
- [ ] Mermaid export parses correctly in mermaid.live
- [ ] If input had identifiable sources, nodes are tagged and sources listed in footer
- [ ] Branch balance: no branch has fewer than 2 or more than 5 sub-items
- [ ] Inquiry nodes (❓) added only where gaps are genuine, not decorative
- [ ] Contradictions marked with ⚡ and red dashed connector, not silently resolved
- [ ] Cross-links limited to 3–4 max, each adds non-obvious insight
- [ ] Hover tooltip renders at a fixed position (top-left), not following the cursor
- [ ] Reading mode detail text truncates at word boundaries, never mid-word
- [ ] Progressive disclosure toggle switches between map mode and reading mode
- [ ] Focus mode isolates a branch and fades the rest to ~8% opacity
- [ ] Focus mode "Back to full map" button works correctly
- [ ] Semantic grouping: related branches are adjacent on the circle
- [ ] Palette selector present with 4 named palettes; switching updates all colors reactively
- [ ] Fullscreen toggle (⛶) works; Escape key exits; body scroll locked while active
- [ ] Weighted nodes are subtly larger, not cartoonishly oversized
- [ ] The map tells a coherent story you could present to someone

## Edge Cases

- **Very short input** (a single sentence or word): Create a brainstorm-style map that explores
  facets of the topic. Add a note: "I expanded this into an exploratory mind map."
- **Very long input** (5000+ words): Summarize aggressively. Aim for the same 4–7 branches with
  3–4 sub-items each. Mention what was compressed.
- **Ambiguous input**: Ask the user for clarification, but offer a default interpretation and
  generate a map from it so they have something to react to. Always generate first, ask second.
- **Lopsided source material**: If the input covers some topics deeply and others shallowly,
  flag the imbalance after generating: "Branches X and Y are thinner because the source
  material doesn't cover them in detail. Want me to research those further?"
- **Non-English input**: Generate the mind map in the same language as the input.
- **Request to update an existing map**: If the user wants to modify a previously generated map,
  regenerate the full artifact with the requested changes applied.

## Example Output Pattern

When responding to the user:

1. Briefly explain what structure you extracted (1–2 sentences)
2. Generate the React artifact
3. Offer to adjust: "Want me to expand any branch, change the structure, or adjust the visual style?"
