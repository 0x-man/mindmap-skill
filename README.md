# 🧠🗺️ Mind Map Generator — Claude Skill

A Claude skill that generates interactive, visually polished mind maps from any content — text, documents, conversations, topics, brainstorms, or ideas.

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

## Installation

### Option A: Install the `.skill` file (easiest)
1. Download `mindmap.skill` from the [Releases](../../releases) page
2. Drag it into any Claude chat, or go to **Settings → Profile → Custom Skills** and upload it

### Option B: Manual install
1. Clone this repo
2. Copy the `mindmap/` folder to your Claude skills directory

## File Structure

```
mindmap/
├── SKILL.md                              # Core instructions + mandatory code (319 lines)
└── references/
    ├── template.md                       # Complete working React template
    ├── mindmap-best-practices.md         # Cognitive science, palettes, keyword compression
    ├── export-patterns.md               # SVG/PNG/PDF/Mermaid/Markdown/Embed export code
    ├── layout-engine.md                 # 4 layout algorithms (radial, semicircle, tree, flow)
    └── atlas-storage.md                 # Persistent storage, atlas viewer, auto-linking
```

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

## License

MIT
