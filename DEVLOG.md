# DEVLOG — Coordinate Studio

Development notes, bug findings, and fixes. Updated as work progresses.

---

## Session 1 — 2026-02-18

### What This Site Is

A personal portfolio site for Chester (mechanical engineer, Denver CO). It has a engineering/CAD aesthetic — ruled grid background, coordinate rulers on the edges, monospace type, and an interactive Conway's Game of Life canvas layered over the whole thing.

**Files:**
| File | Purpose |
|------|---------|
| `index.html` | Main page — GOL canvas, hero card, experience table |
| `design-1.html` | Contact page |
| `design-2.html` | Projects variant (not in main nav) |
| `design-3.html` | About page |
| `design-4.html` | Projects page |
| `Conways-variant.html` | Alternate GOL experiment |

---

### Fix 1 — Nav Menus Were All Dead Links

**Problem:** All nav links in `design-1.html` through `design-4.html` and `Conways-variant.html` pointed to `href="#"`. Only `index.html` had real links.

**Also:** `design-2.html` had a `← RETURN TO INDEX` button also pointing to `#`.

**Label inconsistency:** `design-3.html` used "Agency [C]" while all other pages used "About [C]". Standardized to "About".

**Fix:** Updated all nav links across all files to real destinations:
```
Index [A]    → index.html
Projects [B] → design-4.html
About [C]    → design-3.html
Contact [D]  → design-1.html
```

Also added `class="active"` to the current page's nav item in `design-3.html` (it was missing).

**Commit:** `c1e4c47` — *Connect nav menus across all pages*

---

### Fix 2 — GOL Cells Overlapping Grid Lines (1px offset)

**Problem:** The CSS background grid draws lines at positions `0, 20, 40...` pixels from the element's origin using `background-size: 20px 20px`. The GOL canvas renders cells at `ctx.fillRect(x * CELL, y * CELL, CELL, CELL)` — which starts cells exactly ON the grid lines. The 1px grid lines get painted over by the cell, so:
- The left/top border of each cell is invisible (covered by cell fill)
- The right/bottom border is visible (it's the next cell's left/top)
- Result: cells look asymmetric, like they're shifted 1px left/up in their square

**Fix:** Offset each cell fill by 1px and reduce size by 1px:
```js
// Before
ctx.fillRect(x * CELL, y * CELL, CELL, CELL);

// After
ctx.fillRect(x * CELL + 1, y * CELL + 1, CELL - 1, CELL - 1);
```

Why this works: the CSS grid tile is 20px. The 1px line is at position 0 of each tile. The transparent "interior" of each square spans pixels 1–19 (19px). The new fill starts at position 1 and draws 19px — exactly filling the interior, leaving all four grid lines visible.

Applied to both `index.html` and `Conways-variant.html`.

**Commit:** `dac8195` — *Fix GOL cell alignment with background grid*

---

### Fix 3 — GOL Cells Drifting Radially at Non-100% Zoom

**Problem (the bigger one):** The original `resizeCanvas()` in `index.html` used `devicePixelRatio` to scale the canvas for Retina displays:

```js
function resizeCanvas() {
    const dpr  = window.devicePixelRatio || 1;
    const cssW = window.innerWidth  - OFFSET_X;
    const cssH = window.innerHeight - OFFSET_Y;
    canvas.width  = Math.round(cssW * dpr);
    canvas.height = Math.round(cssH * dpr);
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
    render();
}
```

The intent was correct — multiply by DPR to get physical pixels, then scale the drawing context to compensate. But there were two failure modes:

1. **`window.innerWidth` ≠ actual CSS canvas width:** At non-100% browser zoom levels, the CSS computed value of `calc(100% - 30px)` can differ slightly from `window.innerWidth - 30` due to sub-pixel rounding. The canvas attribute gets set to one size but displayed at a slightly different size, causing the browser to stretch the canvas content. This stretch is proportional to distance from the origin — so cells near the left edge look fine, but cells toward the right drift further right. Left side drifts left, right side drifts right.

2. **`Math.round()` introduces rounding error:** `Math.round(cssW * dpr)` can be off by 1 physical pixel, creating a tiny but accumulating stretch across the full canvas width.

**Symptoms described:** "It almost was like they were rendered on a different layer. They moved with zoom. Ones on the left were left. Right were right."

**Fix:** Drop the DPR scaling entirely. Use `getBoundingClientRect()` to get the actual rendered CSS size of the canvas element (accurate, accounts for any browser rounding), and set canvas dimensions 1:1 with CSS pixels. Add `image-rendering: pixelated` so the browser upscales the canvas content crisply on Retina displays instead of blurring it.

```js
// Before
function resizeCanvas() {
    const dpr  = window.devicePixelRatio || 1;
    const cssW = window.innerWidth  - OFFSET_X;
    const cssH = window.innerHeight - OFFSET_Y;
    canvas.width  = Math.round(cssW * dpr);
    canvas.height = Math.round(cssH * dpr);
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
    render();
}

// After
function resizeCanvas() {
    const rect = canvas.getBoundingClientRect();
    canvas.width  = Math.round(rect.width);
    canvas.height = Math.round(rect.height);
    render();
}
```

```css
/* Added to #golCanvas */
image-rendering: pixelated;
```

**Why `getBoundingClientRect()` is better than `window.innerWidth - OFFSET_X`:**
- `getBoundingClientRect()` returns the actual rendered size of the element as the browser sees it
- No assumptions about what `OFFSET_X` matches in the CSS
- Stays accurate at any zoom level, any screen DPR, any display configuration

**Why removing DPR is fine here:** GOL cells are 20×20px — large enough that the difference between 1x and 2x rendering is invisible. The `pixelated` rendering hint ensures the 1x canvas gets scaled up crisply on HiDPI displays rather than blurred.

Applied to both `index.html` and `Conways-variant.html`.

**Commit:** `5385bc6` — *Fix GOL canvas DPR scaling and alignment*

---

## Architecture Notes

### The Coordinate System

The site is built around a grid system with a 20px cell size (`--cell-size: 20px`). Everything snaps to this grid:
- Rulers on the left (30px wide) and top (20px tall)
- Grid background starts at `top: 20px; left: 30px`
- GOL canvas starts at the same position
- JS constants `OFFSET_X = 30` and `OFFSET_Y = 20` mirror the CSS ruler sizes
- `CELL = 20` mirrors `--cell-size: 20px`

**Important:** If you change the ruler sizes or cell size, you need to update BOTH the CSS variables AND the JS constants. They're currently not linked — a future improvement would be to read `--cell-size` from the computed style in JS.

### GOL Canvas Layering

```
z-index 999  → nav (fixed, top-right)
z-index 200  → coordinates display (fixed, bottom-right)
z-index 101  → corner piece (fixed, top-left)
z-index 100  → rulers (fixed, top and left)
z-index  50  → cursor highlight box
z-index  10  → main content (.main-stage)
z-index   2  → GOL canvas (fixed, transparent background)
z-index   0  → grid background layer
```

The GOL canvas is between the grid and the content. Click/drag events on the grid area create/delete cells. Events on `.content-card` and `nav` are excluded.

### Canvas → Screen Coordinate Mapping

After Fix 3, the mapping is simple:
```
canvas pixel (cx, cy)  →  screen CSS position (cx + 30, cy + 20)
grid cell (gx, gy)     →  canvas pixel (gx * 20 + 1, gy * 20 + 1)
grid cell (gx, gy)     →  screen CSS position (gx * 20 + 31, gy * 20 + 21)
```

Mouse to grid cell:
```js
gx = Math.floor((e.clientX - OFFSET_X) / CELL)  // e.clientX = screen CSS X
gy = Math.floor((e.clientY - OFFSET_Y) / CELL)
```

---

## Known Remaining Issues / Ideas

- `design-2.html` exists in the repo but isn't linked from the main nav (intentional?)
- The JS `CELL` constant and CSS `--cell-size` variable are not linked — risky if someone changes one
- `Conways-variant.html` appears to be an experimental version — unclear if it's meant to be live
- Git committer email is auto-configured from hostname — worth setting a real git identity (`git config --global user.email`)
