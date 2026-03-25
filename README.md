# 🧠🗺️ Mind Map Generator — Claude Skill

A Claude skill that generates interactive, visually polished mind maps from any content — text, documents, conversations, topics, brainstorms, or ideas.

## What It Does

Feed it any content and it produces a **React artifact** with:

- **Radial mind map** with semantic grouping (related branches sit adjacent)
- **Interactive controls**: pan, zoom, collapse/expand, focus mode (branch isolation)
- **Content intelligence**: contradiction detection (⚡), gap analysis (❓), cross-branch links, frequency weighting
- **Progressive disclosure**: toggle between keyword-only map view and full detail reading mode
- **Hover tooltips** with expanded context for compressed labels
- **Source attribution** with clickable links and a footer reference list
- **4 color palettes** switchable at runtime via 🎨 selector
- **6 export formats**: SVG, PNG, PDF, Markdown, Mermaid

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
├── SKILL.md                              # Core instructions (378 lines)
└── references/
    ├── mindmap-best-practices.md         # Cognitive science, palettes, intelligence patterns
    └── export-patterns.md               # SVG/PNG/PDF/Mermaid/Markdown export code
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
| Progressive disclosure | Toggle "⊕ Details" to show/hide expanded text under nodes |
| Palette selector | 🎨 button with 4 named palettes (Bauhaus, Ocean Sunset, Nordic Forest, Pastel Garden) |
| Source linking | 🔗 on hover opens source URL; footer shows numbered reference list |

## License

MIT
