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

Generate interactive mind maps as React (.jsx) artifacts using a template-based approach.

## How This Skill Works

This skill uses a **complete working template** at `references/template.md`. You do NOT
generate the React component from scratch. Instead:

1. Read `references/template.md` — it contains a full working React component
2. Read `references/mindmap-best-practices.md` — for content analysis guidance
3. Analyze the user's input to extract the mind map structure
4. Copy the template EXACTLY, replacing ONLY the `mindmapData` constant at the top
5. Output the result as a .jsx artifact

**On follow-up requests** (edits, palette changes, exports): Skip reading reference files.
Modify only the `mindmapData` block in the existing artifact. Do NOT touch the engine code.

## Content Analysis Strategy

Before building the map, think carefully about structure:

- **Central topic**: One clear noun phrase (2–5 words max).
- **Main branches**: 4–7 primary categories. 5–6 is the sweet spot.
- **Sub-branches**: 2–5 items per main branch. Use keywords, not full sentences.
  See the Keyword Compression section in `references/mindmap-best-practices.md` for
  20 before/after examples of how to compress well.
- **Depth limit**: No deeper than 3 levels (central → branch → sub-branch).

When analyzing long or complex content (articles, documents, book chapters):
- First identify the author's main argument → this becomes the central topic
- Group supporting points into thematic clusters → these become main branches
- Extract key evidence, examples, or details → these become sub-branches
- Discard redundancy ruthlessly — mind maps are about compression, not completeness

When the input is a broad topic (e.g., "machine learning") rather than a specific document:
- Use your knowledge to create a well-structured educational overview
- Prioritize the most important/foundational concepts

### Branch Balance Analysis

After extracting your initial structure, check for imbalance:

- **Thin branches** (only 1 sub-item): Merge into a sibling or flesh out.
- **Fat branches** (6+ sub-items): Split into two. Look for a natural fault line.
- If you can't fix imbalance from the source material, tell the user explicitly.

### Content Intelligence

These turn a mind map from a passive summary into an analytical tool.

**Contradiction detection.** When two sources disagree, mark both nodes with ⚡ prefix
and add a `conflict` field linking the two node IDs. The template renders a red dashed
line between conflicting nodes.

**Gap analysis (Inquiry Nodes).** Where source material is thin on a structurally
important topic, add a node with ❓ prefix and `type: "inquiry"`. The template renders
these with a dashed border.

**Frequency-based weighting.** For multi-source input, track how many sources mention
each concept as a `weight` field (1–5). The template renders heavier nodes slightly larger.

**Cross-branch links.** When ideas in different branches have a meaningful relationship,
add them to the top-level `crossLinks` array. Max 3–4.

## Data Structure Reference

The `mindmapData` constant you replace in the template must follow this schema:

```javascript
const mindmapData = {
  central: "Topic Name",
  // layout: "radial",  // optional: "radial"|"semicircle"|"tree"|"flow"
  // sources: [{ id: "s1", label: "Author 2024", url: "https://..." }],
  // crossLinks: [{ from: "s-0-2", to: "s-3-1", label: "enables" }],
  branches: [
    {
      label: "🎯 Branch Name",    // emoji prefix + 2-5 words
      group: 0,                    // semantic grouping (adjacent branches)
      // sourceId: "s1",           // optional source attribution
      children: [
        {
          label: "Sub-item 1",     // 2-5 words, compressed
          // detail: "Expanded context shown on hover",
          // weight: 3,            // 1-5, frequency across sources
          // sourceId: "s2",
        },
        { label: "❓ Under-explored area", type: "inquiry" },
        { label: "⚡ Claim A conflicts", conflict: "s-2-1" },
      ],
    },
  ],
};
```

### Field reference

- `central`: 2–5 word topic name
- `branches[]`: 4–7 main branches, each with `label`, optional `group`, `sourceId`, `children`
- `children[]`: 2–5 sub-items per branch, each with `label`, optional `detail`, `weight`, `type`, `conflict`, `sourceId`
- `group` (integer): Semantic grouping. Same-group branches sit adjacent on the circle.
- `detail` (string): 1–2 sentence expansion shown on hover. Truncate inline at word boundaries.
- `weight` (1–5): Frequency across sources. Omit for single-source maps.
- `type: "inquiry"`: Renders with dashed border and ❓ prefix.
- `conflict` (node ID): Points to contradicting node. Both get ⚡ prefix.
- `sourceId` (string): Links to a source in the `sources` array.
- `sources[]`: Reference list. Only include for content from identifiable sources.
- `crossLinks[]`: Inter-branch relationships. Max 3–4.
- `layout`: Override auto-detection. See `references/layout-engine.md` for algorithms.

### Source Attribution

When input comes from identifiable sources (articles, papers, URLs), populate `sources`
and tag nodes with `sourceId`. Nodes with a source show 🔗 on hover. The map footer
shows a numbered source list. For knowledge-based maps, omit sources entirely.

## Template Usage

### Generating the artifact

1. Read `references/template.md`
2. Copy the entire JSX code block from the template
3. Replace ONLY the `mindmapData` constant (everything between `/* ━━━ REPLACE THIS DATA BLOCK ━━━ */` and `/* ━━━ ENGINE ━━━ */`) with your extracted content
4. Output as a .jsx artifact

The template includes: edge-anchored connectors (via `edgePoint`), 💾 save button,
palette switcher (4 palettes), fullscreen toggle, pan/zoom, collapse/expand, and hover
tooltips. All of these work automatically — you do not need to implement them.

### What the template provides (do NOT reimplement)

| Feature | In template? |
|---------|-------------|
| Edge-anchored connectors | ✅ via `edgePoint()` + `buildCurve()` |
| 💾 Save button (storage API) | ✅ in toolbar |
| 4 color palettes + runtime switching | ✅ 🎨 button |
| Fullscreen toggle | ✅ ⛶ button |
| Pan/zoom/collapse | ✅ mouse + buttons |
| Hover tooltips (fixed position) | ✅ top-left card |
| Fully opaque pills | ✅ no opacity on fills |
| Semantic gravity (group-based) | ✅ in `buildLayout()` |

### Adding export buttons

If the user asks for export functionality, read `references/export-patterns.md` and add
the export functions to the template. This is the ONLY case where you modify the engine
code — and only by adding new functions and buttons, never by changing existing ones.

## Conversational Editing

After the first mind map is generated, users often want to refine it. The goal: make
edits feel instant and cheap, not like starting over.

### Core Principle

The `mindmapData` object is the single source of truth. On edit requests, change ONLY
the data object and regenerate the artifact. The entire engine code stays identical.
Do NOT re-analyze source content. Do NOT re-read reference files.

### Recognizing Edit Intent

| User says | Operation | What changes |
|---|---|---|
| "Add X under Y" | **Add node** | New child in target branch |
| "Remove X" | **Remove node** | Delete from children |
| "Move X to Y" | **Move node** | Remove from old, add to new parent |
| "Rename X to Y" | **Relabel** | Update label field |
| "Merge X and Y" | **Merge** | Combine children, pick better label |
| "Split X" | **Split** | Two branches, redistribute children |
| "Expand X" | **Deepen** | Add 2–4 sub-items |
| "Simplify" | **Prune** | Remove depth-3 nodes, merge thin branches |
| "Change palette" | **Palette** | Change default palette name |
| "Make it a tree" | **Layout** | Set layout field per `references/layout-engine.md` |
| "Save this map" | **Storage** | Already in template — 💾 button handles this |
| "Show my atlas" | **Atlas** | Generate atlas viewer per `references/atlas-storage.md` |

### Edit Response Pattern

1. State the change in one sentence
2. Regenerate the artifact with updated `mindmapData`
3. Stop. Don't re-describe the map.

### Ambiguous and Multi-Edits

If unclear, make your best interpretation and execute. Show the result, let them redirect.
If an edit breaks a rule (e.g., 8 sub-items), do it but flag: "This branch now has 8
items — want me to split it?"

Multiple changes in one request → one artifact regeneration, not multiple. List changes.

## Edge Cases

- **Very short input**: Brainstorm-style map exploring facets of the topic.
- **Very long input** (5000+ words): Compress to 4–7 branches. Mention what was cut.
- **Ambiguous input**: Generate first with best interpretation, ask second.
- **Lopsided source**: Flag imbalance after generating, offer to research thin branches.
- **Non-English input**: Generate the map in the same language.

## What NOT To Do

- Do NOT generate the React component from scratch — ALWAYS copy the template
- Do NOT modify anything below `/* ━━━ ENGINE ━━━ */` except when adding export functions
- Do NOT respond with a plain text summary or outline
- Do NOT use Mermaid.js, Excalidraw, or any other diagram tool
- Do NOT use opacity < 1 on pill backgrounds
- Do NOT put explanatory prose inside the artifact

## Example Output Pattern

**First generation:**
1. Briefly explain what structure you extracted (1–2 sentences)
2. Generate the React artifact (template with your data)
3. Offer: "Want me to expand any branch, change the structure, or adjust the visual style?"

**Subsequent edits:**
1. State what you changed (1 sentence)
2. Regenerate with updated data
3. Stop.
