# Judge Panel — Deep Structure Mode

An opt-in workflow for producing higher-quality mind map structure on complex inputs.
Instead of single-pass extraction, it generates competing structures, scores them, and
synthesizes the best. Costs roughly 5–7x the tokens of normal generation — use only
when explicitly requested or when the input genuinely warrants it.

## When to Use Deep Mode

Trigger when the user says: "--deep", "deep mode", "think harder", "best possible
structure", "take your time", or "this is important."

Also suggest it proactively (but don't auto-run) when:
- The input is a long, dense document (3000+ words) with competing themes
- The content could plausibly be organized several different ways
- The user is mapping something high-stakes (a thesis, a strategy doc, a research paper)

Suggestion phrasing: "This is a rich document that could be structured several ways.
Want me to run deep mode — I'll generate three competing structures, evaluate them
against each other, and synthesize the strongest one? It takes longer but produces a
noticeably better map."

## The Workflow

Deep mode runs entirely within a single response. You play all roles sequentially —
proposers, judges, synthesizer — thinking through each step explicitly.

### Step 1: Three Proposers

Generate THREE distinct mind map structures for the same content. Force genuine
diversity — do not make three variations of the same idea. Each proposer takes a
different organizing principle:

- **Proposer A — Thematic**: Organize by conceptual themes and topic clusters.
- **Proposer B — Functional**: Organize by process, causation, or how things work
  (inputs → mechanisms → outputs, or problem → analysis → solution).
- **Proposer C — Audience-first**: Organize by what a reader most needs to know first,
  prioritizing by importance and decision-relevance rather than logical structure.

For each proposer, produce a complete branch structure (central topic + 4–7 branches +
sub-items). Keep these compact — labels only, no detail fields yet.

### Step 2: Six-Dimension Scoring

Score each of the three structures on these six dimensions, 1–5 each (30 max):

1. **Coverage** — Does it capture all the important content without gaps?
2. **Balance** — Are branches evenly weighted (no 1-item or 8-item branches)?
3. **Clarity** — Would a newcomer immediately understand the organization?
4. **Insight** — Does the structure reveal non-obvious relationships or patterns?
5. **Compression** — Are labels tight and scannable (keyword discipline)?
6. **Memorability** — Is the structure easy to hold in your head and recall?

Present the scores as a compact table. Be a harsh judge — if all three score 28+,
you're not scoring critically enough. Real structures have real weaknesses.

### Step 3: Synthesizer

Do NOT simply pick the highest scorer. Instead:

1. Identify the single strongest element from each proposal (e.g., "A's thematic grouping
   of the technical branches", "B's cause→effect flow in the middle", "C's decision to
   lead with the practical implications").
2. Construct a fourth structure that combines these strengths.
3. Verify the synthesized structure scores higher than all three originals on the six
   dimensions. If it doesn't, keep the best original instead.

### Step 4: Generate

Build the final artifact (React or Markmap) from the synthesized structure, now adding
detail fields, content intelligence (contradictions, gaps, cross-links), and all the
standard rendering. This is a normal generation — the deep work was in the structure.

## Output Format

Show your work concisely so the user sees the value, but don't overwhelm:

```
I ran deep mode on this. Here's how the three approaches compared:

| Approach     | Coverage | Balance | Clarity | Insight | Compress | Memory | Total |
|--------------|----------|---------|---------|---------|----------|--------|-------|
| A: Thematic  | 5        | 3       | 4       | 3       | 4        | 4      | 23    |
| B: Functional| 4        | 5       | 5       | 4       | 4        | 5      | 27    |
| C: Audience  | 4        | 4       | 5       | 5       | 3        | 4      | 25    |

The synthesized structure takes B's process-driven flow, C's decision to lead with
practical impact, and A's clean technical grouping. Final map below.
```

Then generate the artifact.

## Cost Warning

Deep mode uses significantly more tokens (roughly 5–7x). Only run it when:
- The user explicitly requests it, OR
- The user accepts your proactive suggestion

Never run deep mode silently on a normal request — the user should know they're opting
into a more expensive, slower, higher-quality process.

## Keeping It Honest

The judge panel only adds value if the three proposals are genuinely different and the
scoring is genuinely critical. Failure modes to avoid:

- **Fake diversity**: three structures that are secretly the same. Force different
  organizing principles.
- **Inflated scores**: everything scores 27+. Be harsh. Find real weaknesses.
- **Rubber-stamp synthesis**: just picking the winner instead of combining strengths.
  The synthesized structure must be demonstrably better than any single proposal.
