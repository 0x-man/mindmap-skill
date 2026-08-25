# Mind Map Template

This is a complete working React component. When generating a mind map:

1. Copy this ENTIRE template
2. Replace ONLY the `mindmapData` constant at the top with the extracted content
3. Do NOT modify anything below the `/* ━━━ ENGINE ━━━ */` comment

The template includes: edge-anchored connectors, 💾 save button, palette switcher,
pan/zoom, collapse/expand, hover tooltips, and fully opaque pills.

---

```jsx
import { useState, useRef, useCallback, useMemo, useEffect } from "react";

/* ━━━ REPLACE THIS DATA BLOCK ━━━ */
const mindmapData = {
  central: "Your Topic Here",
  // layout: "radial",  // optional: "radial" | "semicircle" | "tree" | "flow"
  // sources: [{ id: "s1", label: "Author 2024", url: "https://..." }],
  // crossLinks: [{ from: "s-0-2", to: "s-3-1", label: "enables" }],
  branches: [
    {
      label: "🧠 Branch One",
      group: 0,
      // sourceId: "s1",
      children: [
        { label: "Sub-item alpha", detail: "Optional detail text shown on hover." },
        { label: "Sub-item beta" },
        { label: "Sub-item gamma" },
      ],
    },
    {
      label: "⚡ Branch Two",
      group: 0,
      children: [
        { label: "Sub-item delta" },
        { label: "Sub-item epsilon" },
      ],
    },
    {
      label: "🔮 Branch Three",
      group: 1,
      children: [
        { label: "Sub-item zeta" },
        { label: "Sub-item eta" },
        { label: "Sub-item theta" },
      ],
    },
    {
      label: "🛡️ Branch Four",
      group: 1,
      children: [
        { label: "Sub-item iota" },
        { label: "Sub-item kappa" },
      ],
    },
    {
      label: "🎯 Branch Five",
      group: 2,
      children: [
        { label: "Sub-item lambda" },
        { label: "Sub-item mu" },
        { label: "Sub-item nu" },
      ],
    },
  ],
};
/* ━━━ ENGINE — DO NOT MODIFY BELOW THIS LINE ━━━ */

const PALETTES = {
  Bauhaus: ["#D64045","#1D3557","#E9B44C","#2D6A4F","#7B2D8E","#E07A5F","#457B9D"],
  "Ocean Sunset": ["#E76F51","#2A9D8F","#E9C46A","#264653","#F4A261","#287271","#BC4749"],
  "Nordic Forest": ["#5B8C5A","#4A6FA5","#C17C74","#7B6D8D","#D4A574","#4D8B8B","#8B6F47"],
  "Pastel Garden": ["#7EB8DA","#B5C99A","#E8A87C","#9B8EC1","#F2CC8F","#81B29A","#E07A5F"],
};
const PAL_NAMES = Object.keys(PALETTES);
const FONT = "system-ui, -apple-system, sans-serif";

function slugify(t) {
  return t.toLowerCase().replace(/[^a-z0-9]+/g, "-").replace(/(^-|-$)/g, "");
}

function lighten(hex, amt) {
  var a = amt || 0.35;
  var n = parseInt(hex.slice(1), 16);
  return "rgb(" +
    Math.min(255, ((n >> 16) & 255) + Math.round(255 * a)) + "," +
    Math.min(255, ((n >> 8) & 255) + Math.round(255 * a)) + "," +
    Math.min(255, (n & 255) + Math.round(255 * a)) + ")";
}

function getPill(label, depth) {
  var fs = depth === 0 ? 17 : depth === 1 ? 14 : 12;
  var pad = depth === 0 ? 34 : depth === 1 ? 22 : 18;
  return {
    w: Math.max(label.length * fs * 0.62 + pad * 2, 90),
    h: depth === 0 ? 50 : depth === 1 ? 38 : 32,
  };
}

function edgePoint(cx, cy, hw, hh, tx, ty) {
  var dx = tx - cx, dy = ty - cy;
  var dist = Math.sqrt(dx * dx + dy * dy);
  if (dist < 0.01) return { x: cx + hw, y: cy };
  var nx = dx / dist, ny = dy / dist;
  var t = 1.0 / Math.sqrt((nx * nx) / (hw * hw) + (ny * ny) / (hh * hh));
  return { x: cx + nx * t, y: cy + ny * t };
}

function buildCurve(ax, ay, aLabel, aDepth, bx, by, bLabel, bDepth) {
  var pa = getPill(aLabel, aDepth), pb = getPill(bLabel, bDepth);
  var s = edgePoint(ax, ay, pa.w / 2, pa.h / 2, bx, by);
  var e = edgePoint(bx, by, pb.w / 2, pb.h / 2, ax, ay);
  var qx = (s.x + e.x) / 2 + (s.y - e.y) * 0.18;
  var qy = (s.y + e.y) / 2 + (e.x - s.x) * 0.18;
  return "M " + s.x + " " + s.y + " Q " + qx + " " + qy + " " + e.x + " " + e.y;
}

function buildLayout(data) {
  var nodes = [], edges = [], cx = 600, cy = 450;
  nodes.push({ id: "root", label: data.central, x: cx, y: cy, depth: 0, childIds: [], collapsed: false, bi: -1, detail: null });
  var branches = data.branches || [];
  var sorted = branches.map(function(b, i) { return Object.assign({}, b, { oi: i }); });
  sorted.sort(function(a, b) { return (a.group || 0) - (b.group || 0); });
  var groups = [], lastG = null;
  sorted.forEach(function(b, i) { if (b.group !== lastG) { groups.push(i); lastG = b.group; } });
  var gapExtra = 0.14;
  var totalGap = Math.max(0, groups.length - 1) * gapExtra;
  var baseStep = (2 * Math.PI - totalGap) / sorted.length;
  var angle = -Math.PI / 2;
  sorted.forEach(function(branch, i) {
    if (i > 0 && groups.indexOf(i) !== -1) angle += gapExtra;
    var bAngle = angle + (i * 0.05 - 0.08);
    var bx = cx + Math.cos(bAngle) * 260, by = cy + Math.sin(bAngle) * 260;
    var bId = "b-" + branch.oi;
    nodes.push({ id: bId, label: branch.label, x: bx, y: by, depth: 1, childIds: [], collapsed: false, bi: branch.oi, detail: branch.detail || null });
    nodes[0].childIds.push(bId);
    edges.push({ from: "root", to: bId, bi: branch.oi, depth: 1 });
    var kids = branch.children || [];
    kids.forEach(function(child, j) {
      var cc = kids.length;
      var spread = Math.PI * (cc > 4 ? 0.65 : 0.5);
      var sa = bAngle - spread / 2;
      var ca = cc === 1 ? bAngle : sa + (j / (cc - 1)) * spread;
      var sx = bx + Math.cos(ca) * 150, sy = by + Math.sin(ca) * 150;
      var cId = "s-" + branch.oi + "-" + j;
      nodes.push({ id: cId, label: child.label, x: sx, y: sy, depth: 2, childIds: [], collapsed: false, bi: branch.oi, detail: child.detail || null });
      var parent = nodes.find(function(n) { return n.id === bId; });
      if (parent) parent.childIds.push(cId);
      edges.push({ from: bId, to: cId, bi: branch.oi, depth: 2 });
    });
    angle += baseStep;
  });
  return { nodes: nodes, edges: edges };
}

export default function MindMap() {
  var layout = useMemo(function() { return buildLayout(mindmapData); }, []);
  var [nodes, setNodes] = useState(layout.nodes);
  var [palName, setPalName] = useState("Bauhaus");
  var [showPal, setShowPal] = useState(false);
  var [tf, setTf] = useState({ x: 0, y: 0, s: 1 });
  var [drag, setDrag] = useState(false);
  var [ds, setDs] = useState({ x: 0, y: 0 });
  var [hov, setHov] = useState(null);
  var [saveMsg, setSaveMsg] = useState(null);
  var [isFullscreen, setIsFullscreen] = useState(false);
  var svgRef = useRef(null);
  var containerRef = useRef(null);

  var pal = PALETTES[palName];
  var slug = slugify(mindmapData.central);

  var nodesMap = useMemo(function() {
    var m = new Map(); nodes.forEach(function(n) { m.set(n.id, n); }); return m;
  }, [nodes]);

  var hidden = useMemo(function() {
    var s = new Set();
    nodes.forEach(function(n) {
      if (n.collapsed) {
        var stk = n.childIds.slice();
        while (stk.length) { var c = stk.pop(); s.add(c); var ch = nodesMap.get(c); if (ch) ch.childIds.forEach(function(id) { stk.push(id); }); }
      }
    });
    return s;
  }, [nodes, nodesMap]);

  function toggle(id) {
    setNodes(function(p) { return p.map(function(n) { return n.id === id ? Object.assign({}, n, { collapsed: !n.collapsed }) : n; }); });
  }

  var onWheel = useCallback(function(e) {
    e.preventDefault();
    setTf(function(t) { return { x: t.x, y: t.y, s: Math.max(0.2, Math.min(4, t.s * (e.deltaY > 0 ? 0.9 : 1.1))) }; });
  }, []);

  function onDown(e) { if (e.button === 0) { setDrag(true); setDs({ x: e.clientX - tf.x, y: e.clientY - tf.y }); } }
  function onMove(e) { if (drag) setTf(function(t) { return { x: e.clientX - ds.x, y: e.clientY - ds.y, s: t.s }; }); }
  function onUp() { setDrag(false); }

  useEffect(function() {
    var el = svgRef.current;
    if (el) { el.addEventListener("wheel", onWheel, { passive: false }); return function() { el.removeEventListener("wheel", onWheel); }; }
  }, [onWheel]);

  useEffect(function() {
    function onKey(e) { if (e.key === "Escape" && isFullscreen) setIsFullscreen(false); }
    document.addEventListener("keydown", onKey);
    return function() { document.removeEventListener("keydown", onKey); };
  }, [isFullscreen]);

  async function handleSave() {
    var key = "maps:" + slug;
    var existing = null;
    try { existing = await window.storage.get(key); } catch(e) {}
    var now = new Date().toISOString();
    var record = {
      data: mindmapData, palette: palName, layout: "radial",
      createdAt: existing ? JSON.parse(existing.value).createdAt : now,
      updatedAt: now, tags: existing ? JSON.parse(existing.value).tags : [],
    };
    try {
      await window.storage.set(key, JSON.stringify(record));
      setSaveMsg(existing ? "Updated ✓" : "Saved ✓");
    } catch(e) { setSaveMsg("Error"); }
    setTimeout(function() { setSaveMsg(null); }, 2000);
  }

  var vEdges = layout.edges.filter(function(e) { return !hidden.has(e.from) && !hidden.has(e.to); });
  var vNodes = nodes.filter(function(n) { return !hidden.has(n.id); });
  var hovNode = hov ? nodesMap.get(hov) : null;

  function nc(bi, depth) {
    if (depth === 0) return { bg: "#0d1b2a", text: "#fff", accent: "#0d1b2a" };
    var c = pal[bi % pal.length];
    return depth === 1 ? { bg: c, text: "#fff", accent: c } : { bg: lighten(c), text: "#333", accent: c };
  }

  var wrapStyle = isFullscreen
    ? { position: "fixed", top: 0, left: 0, width: "100vw", height: "100vh", zIndex: 10000, display: "flex", flexDirection: "column", background: "#f9f8f6", borderRadius: 0 }
    : { width: "100%", height: "100%", minHeight: 560, display: "flex", flexDirection: "column", background: "#f9f8f6", borderRadius: 12, overflow: "hidden", position: "relative" };

  var btnBase = {
    height: 32, border: "1px solid #ddd", borderRadius: 7, background: "#fff",
    cursor: "pointer", display: "flex", alignItems: "center", justifyContent: "center",
    fontFamily: FONT, fontSize: 13,
  };

  return (
    <div ref={containerRef} style={wrapStyle}>
      {/* ── TOOLBAR ── */}
      <div style={{ display: "flex", alignItems: "center", gap: 5, padding: "8px 12px", borderBottom: "1px solid #e8e6e0", background: "#fff", flexWrap: "wrap" }}>
        <span style={{ fontSize: 15, fontWeight: 700, fontFamily: FONT, marginRight: 8 }}>
          {mindmapData.central}
        </span>
        <div style={{ flex: 1 }} />

        {/* Zoom */}
        <button onClick={function() { setTf(function(t) { return { x: t.x, y: t.y, s: Math.max(0.2, t.s / 1.2) }; }); }} style={Object.assign({}, btnBase, { width: 32, fontSize: 16 })}>−</button>
        <button onClick={function() { setTf({ x: 0, y: 0, s: 1 }); }} style={Object.assign({}, btnBase, { width: 32, fontSize: 12 })}>⊡</button>
        <button onClick={function() { setTf(function(t) { return { x: t.x, y: t.y, s: Math.min(4, t.s * 1.2) }; }); }} style={Object.assign({}, btnBase, { width: 32, fontSize: 16 })}>+</button>

        <div style={{ width: 1, height: 22, background: "#e0e0e0", margin: "0 4px" }} />

        {/* Palette */}
        <div style={{ position: "relative" }}>
          <button onClick={function() { setShowPal(!showPal); }} style={Object.assign({}, btnBase, { width: 36, fontSize: 16 })}>🎨</button>
          {showPal && (
            <div style={{ position: "absolute", top: 38, right: 0, background: "#fff", borderRadius: 10, padding: 8, boxShadow: "0 4px 20px rgba(0,0,0,0.12)", border: "1px solid #e8e6e0", zIndex: 50, minWidth: 170 }}>
              {PAL_NAMES.map(function(name) {
                return (
                  <button key={name} onClick={function() { setPalName(name); setShowPal(false); }}
                    style={{ display: "flex", alignItems: "center", gap: 8, width: "100%", padding: "6px 8px", border: "none", borderRadius: 6, background: palName === name ? "#f0f0f0" : "transparent", cursor: "pointer", fontSize: 11, fontFamily: FONT, fontWeight: palName === name ? 700 : 400, color: "#444" }}>
                    <div style={{ display: "flex", gap: 2 }}>
                      {PALETTES[name].slice(0, 6).map(function(c, ci) {
                        return <div key={ci} style={{ width: 8, height: 8, borderRadius: 4, background: c }} />;
                      })}
                    </div>
                    <span>{name}</span>
                  </button>
                );
              })}
            </div>
          )}
        </div>

        <div style={{ width: 1, height: 22, background: "#e0e0e0", margin: "0 4px" }} />

        {/* Fullscreen */}
        <button onClick={function() { setIsFullscreen(!isFullscreen); if (!isFullscreen) document.body.style.overflow = "hidden"; else document.body.style.overflow = ""; }}
          style={Object.assign({}, btnBase, { width: 32, fontSize: 15 })}>
          {isFullscreen ? "✕" : "⛶"}
        </button>

        <div style={{ width: 1, height: 22, background: "#e0e0e0", margin: "0 4px" }} />

        {/* 💾 SAVE */}
        <button onClick={handleSave}
          style={Object.assign({}, btnBase, {
            padding: "0 16px", fontWeight: 700, fontSize: 13,
            color: saveMsg ? "#2D6A4F" : "#555",
            background: saveMsg ? "#e8f5ee" : "#fff",
            borderColor: saveMsg ? "#2D6A4F" : "#ddd",
          })}>
          {saveMsg || "💾 Save"}
        </button>
      </div>

      {/* ── TOOLTIP ── */}
      {hovNode && hovNode.detail && (
        <div style={{ position: "absolute", top: 56, left: 12, zIndex: 20, maxWidth: 260, background: "rgba(255,255,255,0.97)", backdropFilter: "blur(10px)", borderRadius: 10, padding: "10px 14px", boxShadow: "0 4px 16px rgba(0,0,0,0.08)", border: "1px solid #e8e6e0", fontSize: 12, lineHeight: 1.5, color: "#444", fontFamily: FONT }}>
          <div style={{ fontWeight: 700, fontSize: 11, color: nc(hovNode.bi, hovNode.depth).accent, marginBottom: 4 }}>{hovNode.label}</div>
          {hovNode.detail}
        </div>
      )}

      {/* ── SVG ── */}
      <svg ref={svgRef} viewBox="0 0 1200 900"
        style={{ flex: 1, width: "100%", cursor: drag ? "grabbing" : "grab" }}
        onMouseDown={onDown} onMouseMove={onMove} onMouseUp={onUp} onMouseLeave={onUp}>
        <g transform={"translate(" + tf.x + "," + tf.y + ") scale(" + tf.s + ")"}>

          {/* EDGES — edge-anchored via edgePoint */}
          {vEdges.map(function(edge, i) {
            var fromN = nodesMap.get(edge.from);
            var toN = nodesMap.get(edge.to);
            if (!fromN || !toN) return null;
            var col = edge.bi >= 0 ? pal[edge.bi % pal.length] : "#555";
            var d = buildCurve(fromN.x, fromN.y, fromN.label, fromN.depth, toN.x, toN.y, toN.label, toN.depth);
            return (
              <path key={"e-" + i} d={d} fill="none" stroke={col}
                strokeWidth={edge.depth === 1 ? 2.5 : 1.6}
                strokeOpacity={edge.depth === 1 ? 0.45 : 0.25}
                strokeLinecap="round" />
            );
          })}

          {/* NODES — fully opaque pills */}
          {vNodes.map(function(node) {
            var colors = nc(node.bi, node.depth);
            var pill = getPill(node.label, node.depth);
            var w = pill.w, h = pill.h;
            var isH = hov === node.id;
            var hasK = node.childIds.length > 0 && node.depth > 0;
            var fs = node.depth === 0 ? 17 : node.depth === 1 ? 14 : 12;
            var fw = node.depth === 0 ? 700 : node.depth === 1 ? 600 : 400;

            return (
              <g key={node.id}
                transform={"translate(" + node.x + "," + node.y + ")" + (isH ? " scale(1.05)" : "")}
                onMouseEnter={function() { setHov(node.id); }}
                onMouseLeave={function() { setHov(null); }}
                onClick={function(ev) { ev.stopPropagation(); if (hasK) toggle(node.id); }}
                style={{ cursor: hasK ? "pointer" : "default", transition: "transform 0.15s" }}>

                {isH && (
                  <rect x={-w / 2 - 4} y={-h / 2 - 4} width={w + 8} height={h + 8}
                    rx={node.depth === 0 ? 15 : 22} fill="none"
                    stroke={colors.accent} strokeWidth={1.5} opacity={0.3} />
                )}

                <rect x={-w / 2} y={-h / 2} width={w} height={h}
                  rx={node.depth === 0 ? 13 : 19} fill={colors.bg}
                  filter={node.depth === 0 ? "drop-shadow(0 3px 8px rgba(0,0,0,0.15))" : "none"} />

                <text textAnchor="middle" dy="0.35em" fontSize={fs} fontWeight={fw}
                  fill={colors.text} style={{ fontFamily: FONT, pointerEvents: "none" }}>
                  {node.label}
                </text>

                {hasK && (
                  <text x={w / 2 - 14} dy="0.35em" textAnchor="middle"
                    fontSize={11} fill={colors.text} opacity={0.6}
                    style={{ pointerEvents: "none" }}>
                    {node.collapsed ? "▸" : "▾"}
                  </text>
                )}

                {node.detail && node.depth >= 2 && !isH && (
                  <text x={w / 2 + 2} y={-h / 2 + 4} fontSize={9}
                    fill={colors.accent} opacity={0.6}
                    style={{ pointerEvents: "none" }}>ⓘ</text>
                )}
              </g>
            );
          })}
        </g>
      </svg>

      {/* ── HINT ── */}
      <div style={{ position: "absolute", bottom: 10, left: "50%", transform: "translateX(-50%)", zIndex: 10, fontSize: 11, color: "#999", background: "rgba(255,255,255,0.88)", padding: "4px 14px", borderRadius: 20, whiteSpace: "nowrap", fontFamily: FONT }}>
        Scroll to zoom · Drag to pan · Click branches to collapse · 💾 to save to atlas
      </div>
    </div>
  );
}
```
