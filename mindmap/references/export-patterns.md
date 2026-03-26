# Export Implementation Reference

Complete code patterns for all export functions. The SKILL.md describes *what* each export
does and its UI; this file has the implementation details.

## Font Stack for Exports

All exported SVGs must use a portable font stack that renders reliably across desktop,
mobile, and PDF viewers. Inject this as a `<style>` element in the cloned SVG:

```
text { font-family: Arial, Helvetica, 'Segoe UI', Roboto, sans-serif; }
```

Do NOT use `system-ui` or `-apple-system` in exports — these resolve to nothing on
Android, Windows PDF viewers, and most non-browser contexts. The in-browser rendering
can keep using system-ui since it always resolves there.

## Shared: Preparing the SVG for Export

```javascript
function prepareSvgForExport(svgRef, allNodes) {
  // 1. Clone the live SVG element
  const clone = svgRef.current.cloneNode(true);

  // 2. Reset the transform group — remove user pan/zoom
  const g = clone.querySelector("g");
  g.setAttribute("transform", "translate(0,0) scale(1)");

  // 3. Compute a tight bounding box, add ~120px padding, update viewBox
  const xs = allNodes.map(n => n.x);
  const ys = allNodes.map(n => n.y);
  const pad = 120;
  const minX = Math.min(...xs) - pad * 2;
  const minY = Math.min(...ys) - pad;
  const w = Math.max(...xs) - Math.min(...xs) + pad * 4;
  const h = Math.max(...ys) - Math.min(...ys) + pad * 2;
  clone.setAttribute("viewBox", `${minX} ${minY} ${w} ${h}`);
  clone.setAttribute("width", w * 2);   // 2× for retina
  clone.setAttribute("height", h * 2);
  clone.removeAttribute("style");

  // 4. Inject portable font stack
  const sty = document.createElementNS("http://www.w3.org/2000/svg", "style");
  sty.textContent = `text { font-family: Arial, Helvetica, 'Segoe UI', Roboto, sans-serif; }`;
  clone.insertBefore(sty, clone.firstChild);

  // 5. White background
  const bg = document.createElementNS("http://www.w3.org/2000/svg", "rect");
  bg.setAttribute("x", minX);
  bg.setAttribute("y", minY);
  bg.setAttribute("width", w);
  bg.setAttribute("height", h);
  bg.setAttribute("fill", "#ffffff");
  g.insertBefore(bg, g.firstChild);

  // 6. Replace CSS variable fills with concrete hex
  clone.querySelectorAll("[fill]").forEach(el => {
    const f = el.getAttribute("fill");
    if (f && f.startsWith("var(")) el.setAttribute("fill", "#555555");
  });
  clone.querySelectorAll("text").forEach(el => {
    const f = el.getAttribute("fill");
    if (f && f.startsWith("var(")) el.setAttribute("fill", "#555555");
  });

  return { clone, width: w * 2, height: h * 2 };
}
```

## SVG Export

```javascript
function exportSVG(svgRef, nodes, filename) {
  const { clone } = prepareSvgForExport(svgRef, nodes);
  const svgString = '<?xml version="1.0" encoding="UTF-8"?>\n'
    + new XMLSerializer().serializeToString(clone);
  downloadBlob(new Blob([svgString], { type: "image/svg+xml;charset=utf-8" }), `${filename}.svg`);
}
```

## PNG Export

Render SVG onto offscreen canvas at 2× resolution for retina-crisp output:

```javascript
function exportPNG(svgRef, nodes, filename) {
  const { clone, width, height } = prepareSvgForExport(svgRef, nodes);
  const svgString = new XMLSerializer().serializeToString(clone);
  const url = URL.createObjectURL(new Blob([svgString], { type: "image/svg+xml;charset=utf-8" }));
  const img = new Image();
  img.onload = () => {
    const canvas = document.createElement("canvas");
    canvas.width = width; canvas.height = height;
    const ctx = canvas.getContext("2d");
    ctx.fillStyle = "#ffffff";
    ctx.fillRect(0, 0, width, height);
    ctx.drawImage(img, 0, 0, width, height);
    URL.revokeObjectURL(url);
    canvas.toBlob(blob => downloadBlob(blob, `${filename}.png`), "image/png");
  };
  img.onerror = () => URL.revokeObjectURL(url);
  img.src = url;
}
```

## PDF Export

Uses browser's native print dialog for true vector PDF output:

```javascript
function exportPDF(svgRef, nodes, title) {
  const { clone } = prepareSvgForExport(svgRef, nodes);
  const svgString = new XMLSerializer().serializeToString(clone);
  const printWindow = window.open("", "_blank");
  if (!printWindow) { alert("Please allow popups to export PDF"); return; }
  printWindow.document.write(`<!DOCTYPE html><html><head><title>${title}</title>
    <style>@page{size:landscape;margin:0.5in}body{margin:0;display:flex;
    justify-content:center;align-items:center;min-height:100vh}
    svg{max-width:100%;max-height:100vh}</style></head>
    <body>${svgString}</body></html>`);
  printWindow.document.close();
  printWindow.onload = () => { printWindow.print(); printWindow.close(); };
}
```

## Mermaid Export

```javascript
function exportMermaid(data, filename) {
  let lines = ["mindmap", `  root((${data.central}))`];
  data.branches.forEach(branch => {
    const bLabel = branch.label.replace(/^\p{Emoji_Presentation}\s*/u, "").replace(/[()[\]{}]/g, "");
    lines.push(`    ${bLabel}`);
    (branch.children || []).forEach(child => {
      lines.push(`      ${child.label.replace(/[()[\]{}]/g, "")}`);
      (child.children || []).forEach(detail => {
        lines.push(`        ${detail.label.replace(/[()[\]{}]/g, "")}`);
      });
    });
  });
  downloadBlob(new Blob([lines.join("\n")], { type: "text/plain;charset=utf-8" }), `${filename}.mermaid`);
}
```

## Markdown Outline Export

```javascript
function exportMarkdown(data, filename) {
  let lines = [`# ${data.central}`, ""];
  data.branches.forEach(branch => {
    lines.push(`## ${branch.label}`);
    (branch.children || []).forEach(child => {
      lines.push(`- **${child.label}**${child.detail ? ` — ${child.detail}` : ""}`);
      (child.children || []).forEach(detail => {
        lines.push(`  - ${detail.label}`);
      });
    });
    lines.push("");
  });
  if (data.sources?.length) {
    lines.push("---", "### Sources", "");
    data.sources.forEach((s, i) => lines.push(`${i + 1}. [${s.label}](${s.url})`));
  }
  downloadBlob(new Blob([lines.join("\n")], { type: "text/markdown;charset=utf-8" }), `${filename}.md`);
}
```

## Download Helper

```javascript
function downloadBlob(blob, filename) {
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = filename;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  setTimeout(() => URL.revokeObjectURL(url), 200);
}
```

## Embed HTML Export

Generates a self-contained HTML file with a vanilla JS mind map engine that replicates the
React artifact's interactivity without React or any dependencies. The file can be opened
directly in a browser, hosted on any static server, or embedded via `<iframe>`.

The generated HTML file should include:

1. **The data structure** — the same `mindmapData` object, serialized as a JSON literal
2. **All 4 palettes** — as a constant object, with the currently active palette as default
3. **A vanilla JS layout engine** — same radial algorithm as the React version
4. **SVG rendering** — same curved paths, node styling, and color system
5. **Full interactivity** — pan, zoom, collapse/expand, palette switching, hover tooltips
6. **Responsive sizing** — the SVG fills its container, so it works in any iframe size
7. **Portable font stack** — `Arial, Helvetica, 'Segoe UI', Roboto, sans-serif`

The export function should:

```javascript
function exportEmbed(mindmapData, paletteName, filename) {
  // 1. Serialize the data and active palette into the HTML template
  const dataJSON = JSON.stringify(mindmapData);
  
  // 2. Build a complete HTML document string with inlined CSS + JS
  //    The JS engine is a simplified version of the React component
  //    (~150 lines of vanilla JS that handles layout, rendering, and interaction)
  
  // 3. Download the HTML file
  const blob = new Blob([htmlString], { type: "text/html;charset=utf-8" });
  downloadBlob(blob, `${filename}.html`);
  
  // 4. Also copy an iframe snippet to clipboard for convenience
  const iframeSnippet = `<iframe src="${filename}.html" width="100%" height="600" frameborder="0" style="border:1px solid #eee;border-radius:12px"></iframe>`;
  navigator.clipboard.writeText(iframeSnippet).catch(() => {});
}
```

**Important constraints for the generated HTML:**
- Zero external dependencies — no CDN links, no Google Fonts, no framework imports
- The entire file must work when opened via `file://` (not just `http://`)
- Keep total file size under 30KB — the engine is small, the data is the variable part
- Include a `<meta name="viewport">` tag so it works on mobile
- Background should be white (not transparent) since it'll be embedded in unknown contexts
- Include a small footer line: "Generated by Mind Map Skill for Claude" with a link to
  the GitHub repo (if the user has one configured)

**Button UI:** The embed button uses a `</>` label, sits after the MMD button with a divider.
On click: downloads the HTML file AND flashes "Copied iframe!" for 2 seconds to confirm
the clipboard copy happened.
