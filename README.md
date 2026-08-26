# 🧠 Mind Map Generator — Claude Skill

A Claude skill that generates interactive, visually polished mind maps from any content — text, documents, conversations, topics, brainstorms, or ideas.

[**Live demo →**](https://0x-man.github.io/mindmap-skill/) · [Download `.skill`](../../releases/latest) · MIT

## What It Does

Feed it any content and it produces a **React artifact** with:

- **Radial mind map** with semantic grouping (related branches sit adjacent)
- **Interactive controls**: pan, zoom, collapse/expand, focus mode, fullscreen
- **Content intelligence**: contradiction detection (⚡), gap analysis (❓), cross-branch links, frequency weighting
- **Progressive disclosure**: toggle between keyword-only map view and full detail reading mode
- **Hover tooltips** with expanded context for compressed labels
- **Source attribution** with clickable 🔗 links on individual nodes and a footer reference list
- **4 color palettes** switchable at runtime via 🎨 selector
- **7 export formats**: SVG, PNG, PDF, Markdown, Mermaid, embeddable HTML
- **Conversational editing**: add, remove, move, rename, merge, split nodes through natural language — no full regeneration needed
- **Adaptive layout**: auto-selects radial, semi-circular, top-down tree, or left-to-right flow based on content shape — or override manually
- **Knowledge Atlas**: save maps across sessions, auto-detect shared concepts between them, and explore your growing knowledge graph as a force-directed network
- **Dual output**: React artifact (default) for interactive exploration in Claude, or Markmap `.md` for portable use in VS Code, browsers, and markmap.js.org
- **Deep mode**: say `--deep` on complex documents — generates three competing structures, scores them on six dimensions, and synthesizes the strongest one

## Installation

### claude.ai — install the `.skill` file
1. Download `mindmap.skill` from the [Releases](../../releases) page
2. Drag it into any Claude chat, or go to **Settings → Profile → Custom Skills** and upload it

### Claude Code — install the plugin
```
/plugin marketplace add 0x-man/mindmap-skill
/plugin install mindmap@mindmap-marketplace
```

### Manual install
Clone this repo and copy `skills/mindmap/` into your Claude skills directory.

## File Structure

```
.claude-plugin/
├── plugin.json                           # Claude Code plugin manifest
└── marketplace.json                      # Plugin marketplace catalog

skills/mindmap/                           # Single source of truth
├── SKILL.md                              # Core instructions + mandatory code
└── references/
    ├── template.md                       # Complete working React template
    ├── mindmap-best-practices.md         # Cognitive science, palettes, keyword compression
    ├── export-patterns.md                # SVG/PNG/PDF/Mermaid/Markdown/Embed export code
    ├── layout-engine.md                  # 4 layout algorithms (radial, semicircle, tree, flow)
    ├── atlas-storage.md                  # Persistent storage, atlas viewer, auto-linking
    └── judge-panel.md                    # Multi-agent deep structure mode

docs/                                     # GitHub Pages demo site
```

Both distribution channels read from `skills/mindmap/`. The `.skill` file attached to
each release is built from this same folder — edit once, ship everywhere.

## Usage

Just ask Claude to make a mind map:

- *"Mind map this article"* (paste or upload content)
- *"Map out the key concepts of machine learning"*
- *"Turn this PDF into a visual summary"*
- *"Visualize the structure of this meeting transcript"*

Then build your knowledge graph over time:

- *"Save this map"* — persists across sessions
- *"Show my atlas"* — see all your maps as a connected network
- *"Open my Deep Work map"* — reload a saved map
- *"What connects to my crypto map?"* — discover cross-map links

Or export as a portable Markmap file:

- *"Mind map this article --md"* — generates Markmap directly
- *"Convert to markmap"* — converts an existing React map
- *"Make it portable"* — same as above

For dense documents where structure really matters:

- *"Mind map this paper --deep"* — runs the judge panel for a better structure
- *"Think harder about this one"* — same as above

## Features in Detail

| Feature | What it does |
|---------|-------------|
| Semantic gravity | Groups related branches adjacent on the circle |
| Contradiction detection | Flags conflicting claims with ⚡ and red dashed connectors |
| Inquiry nodes | Adds ❓ nodes where source material has genuine gaps |
| Cross-links | Dotted arcs between nodes in different branches |
| Focus mode | Click ◎ on a branch to isolate it, fading everything else |
| Fullscreen | ⛶ button expands the map to fill the viewport; Escape to exit |
| Progressive disclosure | Toggle "⊕ Details" to show/hide expanded text under nodes |
| Palette selector | 🎨 button with 4 named palettes (Bauhaus, Ocean Sunset, Nordic Forest, Pastel Garden) |
| Source linking | 🔗 on hover opens source URL; footer shows numbered reference list |
| Embed export | `</>` button downloads a self-contained HTML file and copies an iframe snippet |
| Edge anchoring | Connectors start/end at pill edges, not node centers — no text overlap |
| Conversational editing | "Add X under Y", "move Z", "merge these two" — surgical updates without regenerating from scratch |
| Adaptive layout | Auto-selects radial, semi-circular, tree, or flow layout based on content. "Make it a tree" to override. |
| Knowledge Atlas | 💾 Save button is built into every map. "Show my atlas" renders a force-directed network of all your maps with auto-detected concept links. |
| Markmap output | Say "markmap" or "--md" to get portable Markdown that works in VS Code, markmap.js.org, and any browser via npx. |
| Deep mode | Say "--deep" for dense documents. Three proposers, six-dimension scoring, one synthesizer. Slower and pricier, noticeably better structure. |

## Token Usage Estimates

Mind map generation is a single-turn operation. Here's what to expect:

| Component | First map | Follow-ups |
|-----------|-----------|------------|
| **Input: SKILL.md** (always loaded) | ~2,300 | ~2,300 |
| **Input: best-practices reference** | ~2,500 | — (skipped) |
| **Input: export-patterns reference** | ~1,400 | — (skipped) |
| **Input: layout-engine reference** | ~1,500 | — (skipped) |
| **Input: atlas-storage reference** | ~1,800 | — (only when atlas requested) |
| **Input: your content** (text/PDF) | 1,000–5,000 | 200–1,000 |
| **Input: system prompt overhead** | ~1,000 | ~1,000 |
| **Output: response + artifact** | ~5,500 | ~3,000 |
| | | |
| **Total** | **~15,000–19,000** | **~7,000–8,000** |

**In practice:** The hybrid architecture (prose for content analysis, inline code for rendering) keeps SKILL.md lean at ~2,300 tokens. Reference files are loaded only on first generation, then skipped — saving ~55% on follow-ups. On Claude Pro, this is well within normal usage. On the API, expect ~$0.05–0.12 per first map and ~$0.02–0.04 per follow-up.

**What affects cost:**
- Longer input documents → more input tokens (but output stays ~5K regardless — the skill compresses aggressively)
- Follow-ups are significantly cheaper than the first generation
- Asking for structural changes ("add a branch", "merge these two") regenerates the full artifact (~5K output); cosmetic changes ("change palette") are lighter

## Troubleshooting

Claude occasionally drops specific rendering features (save button, edge anchoring, opaque pills) when generating mind maps from scratch. This is a known limitation of LLM code generation — the content analysis and structure extraction work reliably, but visual implementation details can vary between generations.

**Quick fix using inline editing (works every time):**

Claude now supports inline artifact editing. If a feature is missing from your generated mind map:

1. Click the artifact to select it
2. Highlight the area where the feature should be (e.g., the toolbar region)
3. Type your instruction directly, for example:
   - *"Add a 💾 Save button here that saves to window.storage"*
   - *"Make all pill backgrounds fully opaque with fillOpacity={1}"*
   - *"Fix connectors to start at pill edges instead of centers"*

Claude will edit the existing artifact code — which it does reliably, since it's modifying code rather than generating from scratch.

**Features that work reliably on first generation:**
- Radial layout with semantic gravity
- 4–7 branch structure with keyword compression
- Content intelligence (contradictions, gaps, cross-links)
- Palette system and switching
- Pan, zoom, collapse/expand
- Hover tooltips

**Features that may need an inline edit prompt:**
- 💾 Save button
- Edge-anchored connectors (vs center-to-center)
- Fully opaque pills (vs semi-transparent)
- Fullscreen toggle

## License

MIT
