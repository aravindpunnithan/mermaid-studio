# Mermaid Studio

A single-file, zero-build Mermaid diagram editor and renderer with high-resolution PNG and print-quality PDF export. Industrial-utilitarian UI with an IDE aesthetic — dark by default, light on demand, JetBrains Mono with programming ligatures throughout.

Open `index.html` in a modern browser and you're working. No server, no install, no Node.

## Features

**Authoring**

- Live preview with a 600ms debounce — type, see the diagram redraw.
- Six built-in examples (ER, flowchart, sequence, class, Gantt, state) one click away from the source pane.
- Split-pane editor and preview with a draggable divider; stacks vertically on mobile.

**Rendering**

- Mermaid 10.9.0 driven programmatically via `mermaid.render()` (no `startOnLoad`).
- Pan, zoom, fit-to-screen, and reset controls via `svg-pan-zoom`.
- Three diagram fonts — JetBrains Mono (default), system Sans, and Serif — applied to both preview and exports.
- Three themes — Dark (default), Light, and High-contrast — pickable from a dropdown, independent of export styling. High-contrast uses Mermaid's `neutral` base with overrides for black-on-white plus thicker strokes for accessibility.
- Post-render box-fit pass that grows shape geometry (rect, circle, ellipse, polygon — including diamonds and hexagons) so labels never bleed out of their containers, even with monospace fonts that Mermaid's measurement underestimates.

**Export**

- PNG at 1×, 2×, 3× (default), or 4× device-pixel ratio. Transparent, white, or black background — independent of the preview theme.
- PDF on A4, A3, or Letter, with automatic landscape/portrait selection and 15 mm margins.
- Outputs are timestamped: `<prefix>-YYYYMMDD-HHMMSS.png` / `.pdf`.
- The filename prefix is editable in the toolbar (default `diagram`). Illegal filename characters are stripped automatically.
- Files are named so a folder of exports sorts chronologically.

**Diagnostics**

- Live metadata footer: inferred diagram type, SVG dimensions at 1×, and physical print size at 300 DPI.
- Render status indicator with success / error states.
- Clear, human-readable error display when Mermaid source fails to parse.

## How it works

The file is fully self-contained — HTML, CSS, and JavaScript in one document, with CDN-loaded dependencies pinned to specific versions:

- [`mermaid@10.9.0`](https://cdn.jsdelivr.net/npm/mermaid@10.9.0/dist/mermaid.min.js) — diagram rendering
- [`jspdf@2.5.1`](https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js) — PDF generation
- [`svg-pan-zoom@3.6.1`](https://cdn.jsdelivr.net/npm/svg-pan-zoom@3.6.1/dist/svg-pan-zoom.min.js) — preview pan / zoom
- [JetBrains Mono via Google Fonts](https://fonts.googleapis.com/css2?family=JetBrains+Mono) — editor and label typography

The favicon is an inline SVG data URI; everything else the browser needs is fetched once on first load and cached.

### Export pipeline

PNG and PDF exports re-render the diagram with an export-appropriate Mermaid theme (`default` for white/transparent backgrounds, `dark` for black), serialise the resulting SVG, and rasterise it via `Image` → `<canvas>` at the chosen resolution. The PDF path additionally encodes the canvas as JPEG (`quality 0.95`) and embeds it through `jsPDF`, with orientation auto-selected from the diagram's aspect ratio.

Before serialisation, the export SVG is briefly mounted in a hidden offscreen DOM sandbox so the same shape-fitting pass that runs in the preview can apply to the export output. This means a mono diagram that fits correctly on screen also fits correctly in the exported file.

### Canvas-taint handling

Mermaid's `htmlLabels: true` produces `<foreignObject>` SVG content that taints `<canvas>` in Chromium and WebKit, breaking `toDataURL()`. To keep export reliable, the export path re-renders with `htmlLabels: false` (plain SVG `<text>`) while the preview keeps `htmlLabels: true` for accurate box sizing. Two code paths, one config builder.

## Running

Double-click `index.html`. That's it.

If you'd rather serve it (some browsers tighten policies on `file://` URLs):

```sh
# Python 3
python3 -m http.server 8000

# or Node
npx serve .
```

then visit `http://localhost:8000/`.

## Browser support

Tested on the latest two versions of Chrome and Safari. Firefox should work — the only feature with vendor variance is `<foreignObject>` rasterisation, and we side-step that with the dual `htmlLabels` strategy described above.

## Project layout

```
.
├── index.html      # the entire app
├── README.md       # this file
├── UNLICENSE       # public-domain dedication
└── .gitignore
```

No build step, no `node_modules`, no lockfile. Edit `index.html` directly.

## License

Released into the public domain under the [Unlicense](UNLICENSE). Do what you want with it.
