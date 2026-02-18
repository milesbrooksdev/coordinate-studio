# Chester's Portfolio Site — Improvement Plan

_Analysis and recommendations for the Conway's Game of Life portfolio site._

---

## Executive Summary

The site has a strong CAD-aesthetic concept with the live Conway's Game of Life background, but suffers from identity fragmentation (personal portfolio vs. "Coordinate Studio" agency), disconnected navigation, and technical inconsistencies. The grid snapping issue needs a precise fix.

---

## P0 — Must Fix (Blocking Issues)

### 1. Grid Snapping — Cells Don't Align to Visual Grid
**Problem:** Conway's GOL cells are positioned at integer grid coordinates but may render slightly off the CSS grid lines due to:
- Canvas DPR scaling in `index.html` (uses `ctx.setTransform(dpr, 0, 0, dpr, 0, 0)` but fills at pixel coordinates)
- Ruler offset math: cells start at `(gx * CELL, gy * CELL)` but visual grid starts at `OFFSET_X=30px`, `OFFSET_Y=20px`

**Fix:**
```javascript
// In render(), account for the offset in canvas space
function render() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = '#000000';
    for (const key of cells) {
        const [x, y] = parseKey(key);
        // Canvas drawing position matches the CSS grid-layer exactly
        ctx.fillRect(x * CELL, y * CELL, CELL, CELL);
    }
}
```
**Verification:** Click a cell — it should perfectly cover the intersection of four CSS grid lines.

### 2. Dead Navigation Links
**Problem:** Most nav items are `href="#"` — only `index.html` has working cross-links.

**Fix Map:**
| Current | Should Link To |
|---------|----------------|
| design-1.html nav | `index.html`, `design-4.html`, `design-3.html`, `design-1.html` |
| design-3.html nav | Same, plus mark active state |
| design-4.html nav | Same |
| Conways-variant.html | Either remove or integrate |

### 3. Identity Crisis — Decide on Personal vs. Agency
**Problem:** The site oscillates between:
- **Personal:** "Chester | Mechanical Engineer", REP Fitness, PS Audio, 3D printer hobby
- **Agency:** "Coordinate Studio", "NYC_CORE_10013", fake clients (Apex Financial, etc.)

**Decision Required:**
- **Option A:** Full personal portfolio (recommended) — keep Chester's actual work history, add real project case studies from REP/PS Audio if possible, lose the fake agency branding
- **Option B:** Full fictional agency — commit to the Coordinate Studio aesthetic, but this wastes your actual engineering background

---

## P1 — Should Fix (Significant Improvements)

### 4. Conway's GOL Only on Homepage
**Problem:** The interactive GOL is the site's signature — but it's missing from design-1, design-3, and design-4. This breaks the cohesive "CAD interface" metaphor.

**Fix:**
- **Option A:** Add GOL to all pages (extract JS to shared file)
- **Option B:** Keep GOL on homepage only, but add subtle grid-line styling to other pages so they feel related

### 5. No Shared Assets — Code Duplication
**Problem:** Each HTML file repeats ~200 lines of identical CSS and JS for rulers, coordinates, grid styling.

**Fix:**
```
conways-site/
├── css/
│   └── base.css          # Shared: grid, rulers, nav, cards
├── js/
│   └── gol.js            # Shared: Conway engine, render, controls
│   └── grid-ui.js        # Shared: rulers, cursor HUD
├── index.html            # Uses shared + personal content
├── projects.html         # Renamed from design-4.html
├── about.html            # Renamed from design-3.html  
└── contact.html          # Renamed from design-1.html
```

### 6. design-3.html — Complete Aesthetic Outlier
**Problem:** This page uses Inter + JetBrains Mono, thick 4px borders, and a completely different layout philosophy. It looks like a different website.

**Options:**
- **Rewrite** to match the grid/ruler aesthetic of index.html
- **Remove** and merge its content (project table) into a unified projects page

### 7. design-4.html — Hardcoded Dimensions Break Responsiveness
**Problem:** `body { width: 1440px; height: 1024px; }` — this breaks on mobile, tablets, and any non-1440p desktop.

**Fix:** Use the responsive approach from `index.html`:
```css
body {
    width: 100vw;
    min-height: 100vh;
    /* NOT fixed 1440px */
}
```

---

## P2 — Nice to Have (Polish)

### 8. Mobile Experience
**Current:** Crosshair cursor + hover effects don't translate to touch.
**Fixes:**
- Detect touch and disable cursor HUD
- Add tap-to-toggle cells (already works, but test)
- Stack content cards vertically on narrow screens
- Consider hiding GOL on mobile or making it background-only (no interaction)

### 9. Content Enhancements
- **Projects:** Replace placeholder cards (design-4.html) with real REP Fitness or PS Audio work
- **About:** Add photo of you + your 3D printer setup
- **Contact:** Form doesn't actually send — add Netlify Forms, Formspree, or mailto: at minimum

### 10. Visual Polish
- **Active Cell Indicator:** The `cursorBox` sometimes flickers near content cards — refine the `isOverContent()` detection
- **GOL Color:** Consider subtle color variation or the yellow highlight color for live cells
- **Loading State:** Random pixel-art seed can look messy — consider a cleaner startup pattern

### 11. File Cleanup
- `Conways-variant.html` — nearly identical to `index.html`, appears to be a draft. **Delete or rename** to avoid confusion.
- `design-*.html` naming — non-descriptive. Rename to `projects.html`, `about.html`, `contact.html`.

---

## Recommended Action Plan

### Phase 1: Fix the Core (This Weekend)
1. Fix grid snapping math in `index.html`
2. Decide: Personal vs. Agency — pick one, commit
3. Fix all navigation links
4. Make `design-4.html` responsive

### Phase 2: Unify (Next Weekend)
1. Extract shared CSS to `base.css`
2. Extract shared JS to `gol.js` + `grid-ui.js`
3. Rewrite or remove `design-3.html` to match aesthetic
4. Add GOL to all pages OR commit to homepage-only with consistent styling

### Phase 3: Content (Ongoing)
1. Replace placeholder project cards with real work
2. Hook up contact form
3. Mobile testing + touch optimization

---

## Technical Note: The Grid Snapping Fix

The issue is subtle. The canvas and CSS grid **must** agree on:
- Cell size: 20px
- Grid origin: (30px, 20px) — after the rulers

Current code does this correctly for the most part, but verify:
1. Canvas `width`/`height` attributes (not CSS) should be `window.innerWidth - 30` and `window.innerHeight - 20`
2. No extra transforms applied before drawing
3. `ctx.fillRect(x * 20, y * 20, 20, 20)` draws exactly aligned

Test: Pause GOL, click to place one cell, screenshot and verify edges align with CSS grid lines.

---

_Last analyzed: 2026-02-18_
