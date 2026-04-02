# Persistent Storage & Atlas

Mind maps can persist across sessions using the `window.storage` API. This enables
a personal knowledge atlas — a collection of linked maps that grows over time.

## Storage Schema

All maps are stored under a `maps:` prefix. Each map is a single key containing the
full data structure plus metadata.

```javascript
// Save a map
const mapKey = `maps:${slugify(mindmapData.central)}`;
await window.storage.set(mapKey, JSON.stringify({
  data: mindmapData,           // the full mindmapData object
  palette: currentPaletteName, // which palette was active
  layout: currentLayout,       // which layout mode
  createdAt: new Date().toISOString(),
  updatedAt: new Date().toISOString(),
  tags: [],                    // user-defined tags for filtering
}));

// Load a map
const result = await window.storage.get(mapKey);
const saved = JSON.parse(result.value);

// List all maps
const keys = await window.storage.list('maps:');
// keys.keys → ["maps:deep-work", "maps:macro-case-for-crypto", ...]

// Delete a map
await window.storage.delete(mapKey);
```

## Save Button UI

Add a 💾 button to the control bar, after the fullscreen toggle. Behavior:

1. **First save**: Stores the map. Button flashes "Saved ✓" for 2 seconds.
2. **Subsequent saves** (same map title): Overwrites with updated data and new
   `updatedAt` timestamp. Button flashes "Updated ✓".
3. **Auto-save on edit**: When the user makes a conversational edit and the map is
   regenerated, auto-save if the map was previously saved. Don't auto-save on first
   generation — wait for explicit user action.

The save function:

```javascript
async function saveMap(mindmapData, paletteName, layout) {
  const slug = slugify(mindmapData.central);
  const key = `maps:${slug}`;

  let existing = null;
  try { existing = await window.storage.get(key); } catch(e) {}

  const record = {
    data: mindmapData,
    palette: paletteName,
    layout: layout || 'radial',
    createdAt: existing ? JSON.parse(existing.value).createdAt : new Date().toISOString(),
    updatedAt: new Date().toISOString(),
    tags: existing ? JSON.parse(existing.value).tags : [],
  };

  await window.storage.set(key, JSON.stringify(record));
  return existing ? 'updated' : 'saved';
}
```

## Atlas Viewer

The atlas is a separate artifact that the user can request: "Show my atlas", "Show all
my maps", "Open my knowledge graph." It loads all saved maps and renders them as a
network visualization.

### Requesting the Atlas

When the user asks to see their atlas, generate a React artifact that:

1. Loads all maps from storage (keys starting with `maps:`)
2. Parses each map's data to extract branch labels and node labels
3. Detects shared concepts between maps (auto-linking)
4. Renders a force-directed graph where:
   - Each map is a large node (showing the map title and branch count)
   - Shared concepts are edges between maps (labeled with the shared term)
   - Node color comes from the map's saved palette
   - Click a map node to expand it inline (show its branches)
   - Double-click to open the full map (regenerate the mind map artifact)

### Atlas Layout

Use a simple force-directed simulation:

```javascript
function forceLayout(mapNodes, links, width, height) {
  // Initialize random positions
  mapNodes.forEach(n => {
    n.x = width/2 + (Math.random() - 0.5) * width * 0.6;
    n.y = height/2 + (Math.random() - 0.5) * height * 0.6;
    n.vx = 0; n.vy = 0;
  });

  // Run 100 iterations
  for (let iter = 0; iter < 100; iter++) {
    // Repulsion between all nodes
    for (let i = 0; i < mapNodes.length; i++) {
      for (let j = i + 1; j < mapNodes.length; j++) {
        const dx = mapNodes[j].x - mapNodes[i].x;
        const dy = mapNodes[j].y - mapNodes[i].y;
        const dist = Math.max(Math.sqrt(dx*dx + dy*dy), 1);
        const force = 5000 / (dist * dist);
        mapNodes[i].vx -= dx/dist * force;
        mapNodes[i].vy -= dy/dist * force;
        mapNodes[j].vx += dx/dist * force;
        mapNodes[j].vy += dy/dist * force;
      }
    }

    // Attraction along links
    links.forEach(link => {
      const a = mapNodes.find(n => n.id === link.from);
      const b = mapNodes.find(n => n.id === link.to);
      const dx = b.x - a.x, dy = b.y - a.y;
      const dist = Math.sqrt(dx*dx + dy*dy);
      const force = (dist - 200) * 0.01; // rest length 200px
      a.vx += dx/dist * force;
      a.vy += dy/dist * force;
      b.vx -= dx/dist * force;
      b.vy -= dy/dist * force;
    });

    // Apply velocity with damping
    mapNodes.forEach(n => {
      n.x += n.vx * 0.3;
      n.y += n.vy * 0.3;
      n.vx *= 0.8; n.vy *= 0.8;
      // Keep in bounds
      n.x = Math.max(60, Math.min(width-60, n.x));
      n.y = Math.max(60, Math.min(height-60, n.y));
    });
  }
}
```

### Auto-Linking Algorithm

Detect shared concepts between maps by comparing node labels:

```javascript
function findSharedConcepts(maps) {
  const links = [];

  // Build a label → mapId index
  const labelIndex = new Map(); // label → Set of map slugs
  maps.forEach(map => {
    const slug = slugify(map.data.central);
    const allLabels = extractAllLabels(map.data);
    allLabels.forEach(label => {
      // Normalize: lowercase, strip emoji, trim
      const norm = label.replace(/^\p{Emoji_Presentation}\s*/u, '')
                        .toLowerCase().trim();
      if (norm.length < 3) return; // skip tiny labels
      if (!labelIndex.has(norm)) labelIndex.set(norm, new Set());
      labelIndex.get(norm).add(slug);
    });
  });

  // Find labels that appear in 2+ maps
  labelIndex.forEach((mapSlugs, label) => {
    if (mapSlugs.size >= 2) {
      const slugArray = [...mapSlugs];
      for (let i = 0; i < slugArray.length; i++) {
        for (let j = i + 1; j < slugArray.length; j++) {
          // Check if this pair already has a link
          let existing = links.find(l =>
            (l.from === slugArray[i] && l.to === slugArray[j]) ||
            (l.from === slugArray[j] && l.to === slugArray[i])
          );
          if (existing) {
            existing.sharedConcepts.push(label);
          } else {
            links.push({
              from: slugArray[i],
              to: slugArray[j],
              sharedConcepts: [label],
            });
          }
        }
      }
    }
  });

  return links;
}

function extractAllLabels(mindmapData) {
  const labels = [mindmapData.central];
  mindmapData.branches.forEach(b => {
    labels.push(b.label);
    (b.children || []).forEach(c => {
      labels.push(c.label);
      (c.children || []).forEach(d => labels.push(d.label));
    });
  });
  return labels;
}
```

### Atlas UI Components

The atlas artifact should include:

**Header bar:**
- Title: "Knowledge Atlas" with map count (e.g., "12 maps, 8 connections")
- Search/filter input to highlight maps by keyword
- Sort: by date created, date updated, or connection count

**Map nodes (the circles in the graph):**
- Size proportional to branch count (more branches = larger circle)
- Color from the map's saved palette (first color)
- Label: map title inside or below the circle
- Subtitle: "6 branches · Updated 2d ago"
- Hover: show branch labels as a mini-list tooltip
- Click: expand inline to show branches (like a mini mind map)
- Right-click or long-press: options menu (Open, Delete, Edit tags)

**Connection edges:**
- Thin lines between connected maps
- Label at midpoint: shared concept count (e.g., "3 shared")
- Hover the edge: show the list of shared concepts
- Thicker line = more shared concepts

**Empty state:**
- "No maps saved yet. Generate a mind map and click 💾 to save it here."
- Suggest example prompts to get started

**Stats sidebar (collapsible):**
- Total maps, total nodes across all maps, total connections
- Most connected map (your "hub" topic)
- Isolated maps (no connections — might need more maps to link)
- Recently updated maps

### Atlas Interaction

- Pan and zoom (same as mind maps)
- Drag map nodes to manually reposition
- Click a connection edge to see shared concepts
- "Open in map view" button on each node to generate the full mind map

## Loading a Saved Map

When the user asks to open a specific map ("open my Deep Work map", "load the crypto
mind map"), generate the mind map artifact using the saved data:

```javascript
async function loadMap(slug) {
  try {
    const result = await window.storage.get(`maps:${slug}`);
    const saved = JSON.parse(result.value);
    return saved; // { data, palette, layout, createdAt, updatedAt, tags }
  } catch(e) {
    return null; // map not found
  }
}
```

Use `saved.data` as the `mindmapData`, `saved.palette` as the default palette,
and `saved.layout` as the layout mode. The mind map renders exactly as it was saved.

## User Commands

| User says | Action |
|-----------|--------|
| "Save this map" | Save current map to storage |
| "Show my atlas" / "show all maps" | Generate the atlas viewer artifact |
| "Open my [topic] map" | Load saved map and generate mind map artifact |
| "Delete my [topic] map" | Delete from storage, confirm first |
| "How many maps do I have?" | List saved maps with metadata |
| "What connects to my [topic] map?" | Show maps with shared concepts |
| "Tag this map as [tag]" | Update the tags array in storage |
