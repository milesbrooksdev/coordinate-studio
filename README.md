# Coordinate Studio — Interactive Portfolio

A technical/CAD-aesthetic portfolio website featuring a live, interactive **Conway's Game of Life** simulation running in the background. Built with vanilla HTML, CSS, and JavaScript — no dependencies, no build step.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?logo=javascript&logoColor=black)

---

## Live Demo

Open `index.html` in any modern browser. The Conway's Game of Life simulation starts automatically — click and drag on the grid to paint cells, use the controls to pause, step, clear, or reseed.

---

## Concept

The web is a grid, not a canvas. This site exposes that underlying structure:

- **Live grid background** with ruler axes (like CAD software)
- **Interactive GOL simulation** — click or drag to paint cells
- **Coordinate HUD** — real-time X/Y position tracking
- **Brutalist/technical aesthetic** — exposed borders, monospace typography, functional design

The Game of Life isn't just decoration — it's a computational system running live, symbolizing how simple rules create complex outcomes (much like engineering design).

---

## Structure

```
conways-site/
├── index.html              # Main portfolio (Chester / Coordinate Studio)
├── Conways-variant.html    # Alternate styling exploration
├── design-1.html           # Contact page
├── design-3.html           # About / Agency page (alternate aesthetic)
├── design-4.html           # Projects archive
├── README.md               # This file
└── IMPROVEMENT_PLAN.md     # Detailed roadmap for enhancements
```

---

## Features

### Conway's Game of Life Implementation
- **Canvas-based rendering** at 60fps
- **Optimized neighbor counting** using a Map for O(n) performance
- **Interactive painting** — click to toggle, drag to paint/erase
- **Controls:** Play/Pause, Step (single generation), Clear, Reseed (random pixel art)
- **Live statistics** — generation count and active cells

### CAD-Style UI Elements
- **Ruler axes** — Excel-style column labels (A, B, C... AA, AB) + row numbers
- **Active cell indicator** — follows cursor, snaps to grid
- **Coordinate display** — real-time X/Y position in bottom-right
- **Crosshair cursor** throughout

### Responsive-ish Layout
- CSS Grid-based content cards
- Percentage-based positioning relative to the grid
- Fixed cell size (20px) for visual consistency

---

## How It Works

### The Grid System

The site uses a strict 20px grid system:

```css
:root {
    --cell-size: 20px;
    --offset-x: 30px;    /* ruler-y width */
    --offset-y: 20px;    /* ruler-x height */
}
```

All content cards are positioned using `calc(var(--cell-size) * N)` to maintain alignment.

### Conway's GOL Engine

```javascript
// Cells stored as "x,y" strings in a Set
const cells = new Set();

// Render: each cell fills exactly one 20×20 grid cell
function render() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = '#000000';
    for (const key of cells) {
        const [x, y] = parseKey(key);
        ctx.fillRect(x * CELL, y * CELL, CELL, CELL);
    }
}

// Step: standard Conway rules
// - Live cell with 2-3 neighbors survives
// - Dead cell with exactly 3 neighbors becomes alive
// - All others die
```

### Interaction Model

The canvas uses `pointer-events: none` so clicks pass through to the document handler. The script calculates grid coordinates from mouse position:

```javascript
const gx = Math.floor((e.clientX - OFFSET_X) / CELL);
const gy = Math.floor((e.clientY - OFFSET_Y) / CELL);
```

This allows seamless interaction between the GOL layer and the content cards above it.

---

## Browser Support

- Chrome/Edge 90+
- Firefox 88+
- Safari 14+

Requires:
- CSS Grid support
- Canvas 2D context
- ES6 (const, arrow functions, Set)

---

## Known Issues & TODO

See `IMPROVEMENT_PLAN.md` for detailed analysis. High-level:

1. **Navigation links** — Some pages use placeholder `href="#"` links
2. **Code duplication** — CSS/JS repeated across files; should extract to shared modules
3. **Aesthetic consistency** — `design-3.html` uses a completely different visual language
4. **Mobile experience** — Fixed dimensions on some pages break on mobile
5. **Grid alignment** — Cells should snap perfectly to CSS grid lines (subpixel precision)

---

## Design Philosophy

> "The web is a grid, not a canvas. By exposing the underlying logic of the browser, we create interfaces that feel honest, raw, and utilitarian."

This site rejects decorative excess in favor of:
- **Structural honesty** — borders, margins, and spacing are visible and purposeful
- **Functional density** — every element serves a purpose
- **Computational presence** — the live GOL is not decoration, it's a running system

---

## License

MIT — feel free to fork, remix, or use as a template. Attribution appreciated but not required.

---

## Credits

- **Conway's Game of Life** — [John Horton Conway](https://en.wikipedia.org/wiki/John_Horton_Conway) (1937–2020)
- **Design & Code** — Chester / Coordinate Studio

---

*Built with vanilla everything. No frameworks, no build step, no nonsense.*
