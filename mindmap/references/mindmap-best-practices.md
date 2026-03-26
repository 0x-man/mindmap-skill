# Mind Map Best Practices Reference

## Cognitive Science Behind Mind Maps

Mind maps work because they mirror how the brain organizes information — through association,
not linear sequence. Research on dual coding theory (Paivio, 1986) shows that combining visual
and verbal information improves recall by 65% compared to text alone. The radial structure
exploits the brain's natural pattern-recognition: each branch becomes a retrieval cue.

Key principles backed by cognitive research:

- **Chunking**: Group related items (Miller's 7±2 rule applies — 5–7 branches is optimal)
- **Color coding**: Distinct colors create separate memory channels per branch
- **Spatial positioning**: Where something is on the map becomes part of its memory encoding
- **Keywords over sentences**: Single words or short phrases force distillation, which deepens processing
- **Hierarchy**: Clear parent-child relationships reduce cognitive load

## Content Extraction Heuristics

### From articles/documents:
1. Title/headline → central topic (reword if too long)
2. Section headings → main branches
3. Topic sentences of each section → sub-branches
4. Key statistics, quotes, examples → detail nodes (use sparingly)

### From conversations/meetings:
1. Meeting subject → central topic
2. Agenda items or discussion themes → main branches
3. Decisions made → sub-branches (prefix with ✅)
4. Action items → detail nodes (prefix with 📌)
5. Open questions → detail nodes (prefix with ❓)

### From brainstorming:
1. Core question/challenge → central topic
2. Thematic clusters → main branches (group related ideas)
3. Individual ideas → sub-branches
4. Don't filter — capture everything, let the user prune

### From educational/technical content:
1. Subject name → central topic
2. Core concepts/pillars → main branches
3. Definitions, formulas, key facts → sub-branches
4. Examples → detail nodes
5. Arrange clockwise: foundational → intermediate → advanced

## Emoji Guide for Branch Prefixes

Use ONE emoji per branch. Pick the most semantically meaningful:

| Domain | Common Emojis |
|--------|--------------|
| Goals/Strategy | 🎯 🏆 🚀 📈 |
| Problems/Risks | ⚠️ 🔥 💀 🚧 |
| Ideas/Innovation | 💡 ✨ 🧪 🔮 |
| People/Teams | 👥 🤝 👤 🧑‍💻 |
| Time/Process | ⏰ 📅 🔄 ⚡ |
| Money/Resources | 💰 📊 🏦 💎 |
| Tools/Tech | 🔧 ⚙️ 🖥️ 📱 |
| Learning/Knowledge | 📚 🧠 🎓 📝 |
| Communication | 💬 📣 ✉️ 📡 |
| Security/Privacy | 🔒 🛡️ 🔐 👁️ |
| Health/Wellness | ❤️ 🏥 🧘 💊 |
| Nature/Environment | 🌍 🌱 🌊 ♻️ |
| Creative/Design | 🎨 🖌️ ✏️ 🎭 |
| Food/Cooking | 🍽️ 🥘 🍳 🌿 |
| Travel/Places | ✈️ 🗺️ 🏔️ 🌆 |

## Color Palettes

### Palette 1: "Ocean Sunset" (warm-cool contrast)
```
#E76F51  // Burnt sienna (warm)
#2A9D8F  // Teal (cool)
#E9C46A  // Saffron (warm)
#264653  // Charcoal teal (cool)
#F4A261  // Sandy brown (warm)
#287271  // Deep teal (cool)
#BC4749  // Crimson (warm)
```

### Palette 2: "Nordic Forest" (muted, sophisticated)
```
#5B8C5A  // Sage green
#4A6FA5  // Steel blue
#C17C74  // Dusty rose
#7B6D8D  // Muted purple
#D4A574  // Warm tan
#4D8B8B  // Dark teal
#8B6F47  // Warm brown
```

### Palette 3: "Bauhaus" (bold, confident)
```
#D64045  // Red
#1D3557  // Navy
#E9B44C  // Gold
#2D6A4F  // Forest green
#7B2D8E  // Purple
#E07A5F  // Coral
#457B9D  // Blue gray
```

### Palette 4: "Pastel Garden" (soft, approachable)
```
#7EB8DA  // Sky blue
#B5C99A  // Sage
#E8A87C  // Peach
#9B8EC1  // Lavender
#F2CC8F  // Butter
#81B29A  // Mint
#E07A5F  // Salmon
```

Rotate through palettes across generations to avoid visual monotony. Pick the default
palette that best matches the content's tone: professional/technical → Nordic or Bauhaus,
creative/personal → Ocean Sunset or Pastel Garden. All four palettes ship in every
generated mind map and the user can switch between them at runtime via the 🎨 selector.

## Anti-Patterns to Avoid

1. **Wall of text nodes** — If a node label is more than 5 words, you haven't distilled enough
2. **Too many branches** — More than 7 main branches overwhelms the visual. Merge or group.
3. **Unbalanced trees** — One branch with 12 sub-items and another with 1 looks broken. Redistribute.
4. **No color coding** — Monochrome maps lose 50% of their utility
5. **Straight lines** — Curved organic connections are easier to trace and more visually appealing
6. **Tiny text** — If users need to zoom to 200% to read anything, the layout needs work
7. **Center overload** — Don't put too much info in the central node. It's a label, not a summary.
8. **Uniform node sizes** — Visual hierarchy matters. Make important things bigger.
9. **Decorative clutter** — Emojis and colors serve cognition, not decoration. Every element earns its place.
10. **Fabricated sources** — Only tag nodes with sourceId when the source genuinely supports that node. Never invent references to make the map look more authoritative.

## Source Attribution

When a mind map is generated from specific, identifiable content (an article, paper, book,
URL, or uploaded document), tracing ideas back to their origin adds significant value —
especially for research, study, and professional use.

### When to include sources:
- User pastes or uploads specific content → always attribute
- User provides URLs → always attribute
- User asks to map a book/paper by name → attribute if you can identify sections
- User asks for a general topic map from your knowledge → do NOT attribute (no fabricated refs)

### How sources appear in the map:
- **Footer list**: A small numbered source list below the map canvas (e.g., "[1] Newport 2016")
- **Node hover**: Nodes with a sourceId show a subtle 🔗 icon on hover. Clicking it opens the
  source URL in a new tab.
- **Keep it unobtrusive**: Sources should inform, not clutter. The map itself stays clean;
  attribution lives one interaction layer deeper (hover/click).

## Content Intelligence Patterns

Mind maps are often treated as passive summaries. The features below turn them into
analytical tools that surface the interesting structure in the content — contradictions,
gaps, consensus, and hidden connections.

### When to use each feature

| Feature | Use when... | Don't use when... |
|---------|-------------|-------------------|
| Contradiction (⚡) | Two sources genuinely disagree on a factual claim | It's merely a difference in emphasis or framing |
| Inquiry node (❓) | A structurally important area has no supporting evidence | You just want to pad a thin branch |
| Cross-link | A concept in one branch causally relates to another | The relationship is obvious from the hierarchy |
| Weight | Multi-source input where frequency signals consensus | Single-source or knowledge-based maps |
| Detail tooltip | The 5-word label loses critical nuance | The label is already self-explanatory |

### Contradiction detection heuristics

Common contradiction patterns to watch for when synthesizing multiple sources:
- **Numerical disagreements**: Source A says "30% improvement" vs Source B says "marginal effect"
- **Causal direction**: Source A says X causes Y, Source B says Y causes X
- **Scope claims**: Source A says "universally applicable", Source B says "only in context Z"
- **Recommendation conflicts**: Source A advocates strategy X, Source B warns against it

Don't flag differences in *depth* as contradictions. One source covering a topic in more
detail than another is normal — that's what weight is for.

### Cross-link relationship labels

Keep labels to 1–2 words. Common patterns: "enables", "blocks", "requires", "similar to",
"opposite of", "part of", "leads to", "competes with". The label should make the
relationship self-explanatory without needing the surrounding context.

## Keyword Compression

The 5-word label limit is the single most violated rule. The problem isn't that Claude
can't count to five — it's that compressing a nuanced idea into a scannable keyword
is a learned craft, not an obvious operation. This section teaches it by example.

### Compression Principles

1. **Delete the verb.** Most ideas can be expressed as noun phrases. "The process of
   managing distributed systems" → "Distributed systems mgmt"
2. **Cut articles and prepositions.** "The impact of inflation on savings" → "Inflation
   erodes savings"
3. **Use symbols when obvious.** "is greater than" → ">", "leads to" → "→", "versus" → "vs"
4. **Prefer the specific over the abstract.** "Various types of risk" → "Market + credit risk"
5. **Use the field's own jargon.** Experts scan faster when they see domain terms.
   "The process of making things smaller" → "Minification" (in web dev context)
6. **Put the most important word first.** Scanners read the first word and skip the rest
   if it doesn't match. "Global supply chain disruption" → "Supply chain disruption"
   (not "Disruption of supply chains")
7. **Move nuance to the `detail` field.** The label is a headline; the detail is the
   paragraph. If compression loses critical meaning, that's what `detail` is for.

### Before/After Examples

Study these. The pattern is: verbose input → compressed label (+ detail field when nuance matters).

**Business & Strategy:**
| Input | Label | Detail (if needed) |
|-------|-------|--------------------|
| The company's annual revenue has been declining for three consecutive years | Revenue declining 3 yrs | |
| Customers are increasingly switching to competitor products | Customer churn rising | |
| The organization needs to reduce operational costs by at least 20% | Cut opex ≥20% | |
| Establishing strategic partnerships with key industry players | Strategic partnerships | |
| The risk of regulatory changes affecting our core business model | Regulatory risk to model | Specific regulations could restrict core revenue streams |

**Technology:**
| Input | Label | Detail (if needed) |
|-------|-------|--------------------|
| The application programming interface that connects the frontend to the backend | Frontend ↔ Backend API | |
| Implementing continuous integration and continuous deployment pipelines | CI/CD pipeline | |
| The system experienced a 99.9% uptime rate over the past quarter | 99.9% uptime (Q3) | |
| Migrating the existing infrastructure from on-premises servers to cloud-based solutions | On-prem → cloud migration | |
| The technical debt accumulated from years of rapid feature development | Tech debt accumulation | Years of shipping features without refactoring |

**Science & Research:**
| Input | Label | Detail (if needed) |
|-------|-------|--------------------|
| The experiment demonstrated a statistically significant correlation between variables | Significant correlation found | p < 0.05 across all treatment groups |
| The process by which the body's immune system attacks its own healthy tissue | Autoimmune response | |
| Carbon dioxide concentrations in the atmosphere have reached 420 parts per million | CO₂ at 420 ppm | |
| The phenomenon where observation changes the behavior of the thing being observed | Observer effect | |
| Neurons that fire together strengthen their synaptic connections over time | Hebbian learning | "Neurons that fire together wire together" |

**Finance & Economics:**
| Input | Label | Detail (if needed) |
|-------|-------|--------------------|
| The Federal Reserve's decision to maintain interest rates at their current level | Fed holds rates steady | |
| Government debt as a percentage of gross domestic product has reached 100% | Debt ~100% of GDP | |
| The yield curve has inverted, which historically precedes economic recessions | Yield curve inverted | Historically precedes recession by 12–18 months |
| Investors are moving capital from equities into safer fixed-income instruments | Flight to bonds | Risk-off sentiment driving equity outflows |
| The annual rate at which the general level of prices for goods and services is rising | Inflation rate | |

### Common Compression Failures

These patterns produce bad labels — watch for them:

| Failure | Example | Why it's bad | Better |
|---------|---------|-------------|--------|
| Just truncating | "The process of mana…" | Unreadable | "Process management" |
| Too abstract | "Key considerations" | Says nothing | "Cost vs speed tradeoff" |
| Redundant with parent | Parent: "Risks", Child: "Risk factors" | Wastes a node | "Market volatility" |
| Counting words wrong | "CI/CD pipeline setup and config" (6 words) | Over limit | "CI/CD setup" |
| Emoji overuse | "🔥💰📈 Revenue growth" | Cluttered | "📈 Revenue growth" (one per branch, not per node) |
