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
├── SKILL.md                              # Core instructions (450 lines)
└── references/
    ├── mindmap-best-practices.md         # Cognitive science, palettes, intelligence patterns
    └── export-patterns.md               # SVG/PNG/PDF/Mermaid/Markdown/Embed export code
```

## Usage

Just ask Claude to make a mind map:

- *"Mind map this article"* (paste or upload content)
- *"Map out the key concepts of machine learning"*
- *"Turn this PDF into a visual summary"*
- *"Visualize the structure of this meeting transcript"*

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

## Token Usage Estimates

Mind map generation is a single-turn operation. Here's what to expect:

| Component | Tokens (approx) |
|-----------|-----------------|
| **Input: SKILL.md** (always loaded) | ~5,100 |
| **Input: best-practices reference** (read on first use) | ~1,500 |
| **Input: export-patterns reference** (read on first use) | ~1,400 |
| **Input: your content** (the text/PDF being mapped) | 1,000–5,000 |
| **Input: system prompt overhead** | ~1,000 |
| **Output: response text** (brief explanation) | ~200 |
| **Output: React artifact** (the mind map code) | ~5,000–5,500 |
| | |
| **Total per generation** | **~15,000–20,000** |

**In practice:** A typical mind map costs roughly one mid-length Claude response. On Claude Pro, this is well within normal usage. On the API, expect ~$0.05–0.15 per map depending on your model and input length.

**What affects cost:**
- Longer input documents → more input tokens (but the output stays ~5K regardless — the skill compresses aggressively)
- Asking for revisions ("expand this branch", "change the palette") costs less than the initial generation since the skill is already in context
- The two reference files are only read on the first generation in a conversation — subsequent maps in the same chat skip them

## License

MIT
