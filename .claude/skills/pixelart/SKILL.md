---
name: pixelart
description: Generate pixel art as self-contained HTML files. Pure code, zero dependencies. Supports 8x8 icons to full landscape scenes, game tilesets, and procedural assets.
argument-hint: [size] [prompt] or tileset [theme]
---

Generate a pixel art HTML file based on the user's request: $ARGUMENTS

### Input Modes

**Mode 1 — Single asset (default):**
Parse input to extract **size** (e.g. `16x16`, `32x32`, `192x128`) and **prompt**. If size is omitted, default to `16x16`.

```
/pixelart 32x32, a wizard with a glowing staff
/pixelart 80x85, stone cottage with red tile roof
/pixelart 50x40, a fox sitting in grass
```

**Mode 2 — Tileset:**
When prompt contains **"tileset"**, **"sprite sheet"**, or **"asset pack"**, generate a complete sheet of related assets on a single canvas.

```
/pixelart tileset, medieval village
/pixelart tileset, dungeon crawler
/pixelart tileset, forest nature pack
/pixelart tileset, farm animals
```

---

## Output

A single, complete, self-contained `.html` file. No external images or dependencies.

---

## Auto-Decision Rules

### Size → Detail Level

| Size             | Detail  | Max Colors | Shading                                |
|-----------------|---------|-----------|----------------------------------------|
| 8×8             | minimal | 3–5       | Flat colors, no shading                 |
| 12×12           | standard| 5–8       | 1 shadow tone per material              |
| 16×16           | standard| 8–12      | 1 shadow tone per material              |
| 16×24, 24×24    | detailed| 10–16     | Highlight + base + shadow per material  |
| 32×32+          | detailed| 12–20     | Highlight + base + shadow, texture hints|
| 48×48+          | ultra   | 15–25     | Multi-layer shading, dithering          |
| 96×64+ (canvas) | ultra   | Unlimited | Procedural FBM noise, Bayer dithering   |
| 48×96+ (asset)  | ultra   | Unlimited | Procedural texture engine, region-based  |

### Prompt → Rendering Method

| Prompt implies...              | Method                                 |
|-------------------------------|----------------------------------------|
| Simple icon / item / emoji     | **CSS box-shadow** (zero JS, static)   |
| Character / sprite             | **CSS Grid + JS** (per-pixel control)  |
| "animated", movement, effects  | **Grid + animation JS**                |
| Landscape, scene, environment  | **Canvas** (procedural generation)     |
| Building, tileset, game asset  | **Canvas** (procedural texture engine) |
| Animal, creature, organic      | **Canvas** (procedural organic textures)|

### Prompt → Auto Animations & Atmosphere

| Content in prompt        | Auto-add                                      |
|-------------------------|-------------------------------------------------|
| Character with weapon    | `breathe` + `gleam` (weapon shine)              |
| Character with hair/cloth| `breathe` + `wind` (flutter)                    |
| Has eyes                 | `blink` (periodic)                              |
| Night / dark scene       | Stars, moon, mist                               |
| Fire / lava              | Ember particles, glow                           |
| Water / lake / ocean     | Water reflection animation                      |
| Rain / storm             | Rain particles                                  |
| Cherry blossom / spring  | Falling petals                                  |
| Snow / winter            | Snowflake particles                             |
| Magic / glow / neon      | Glow pulse on emissive pixels                   |
| City / urban             | Building silhouettes, lit windows               |
| Any character            | `fade-in` (staggered pixel reveal)              |

If nothing suggests effects, output static art.

---

## Output Formats

### Small art (≤48×48): String-array data + HTML renderer

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>{{name}} · Pixel Art</title>
<style>
  * { margin:0; padding:0; box-sizing:border-box; }
  body { min-height:100vh; display:flex; align-items:center; justify-content:center;
         background:#1a1a2e; font-family:'Courier New',monospace; }
  .pixel-grid { display:grid; gap:0; image-rendering:pixelated; }
  /* animation keyframes as needed */
</style>
</head>
<body>
<script>
const ART = {
  name: '{{name}}',
  width: {{W}},
  height: {{H}},
  palette: {
    '.': 'transparent',
    // semantic single-char keys → hex colors
  },
  pixels: [
    // each string = one row, each char = one pixel
  ]
};
// Render with CSS Grid or box-shadow
// Add animations based on auto-decision rules
</script>
</body>
</html>
```

### Large art (≥64px): Canvas-based procedural generation

Use `<canvas>` with pixel buffer, FBM noise engine, ordered dithering, and layered rendering:

```javascript
// Core engine
function hash2D(x, y) { /* integer hash → 0-1 */ }
function valueNoise(x, y) { /* smoothstep interpolated */ }
function fbm(x, y, octaves) { /* fractal Brownian motion */ }
function ridgeNoise(x, y, octaves) { /* sharp ridge lines */ }

// Bayer 4×4 ordered dithering
const BAYER = [[0,8,2,10],[12,4,14,6],[3,11,1,9],[15,7,13,5]];
function ditherColor(x, y, c1, c2, ratio) { /* pick c1 or c2 based on threshold */ }

// PixelBuffer class with: set, get, blend, tri, rect, line
// Layer functions: drawSky, drawSun, drawStars, drawMountains, drawTrees, drawWater, drawBuildings, drawMist...
// Render to canvas with imageSmoothingEnabled = false
```

---

## Design Rules

### Palette
- `.` = transparent (always)
- Semantic mapping: `S` skin, `H` hair, `E` eyes, `B` body, `M` mouth, `A` armor, `G` gold, `R` red, `W` white, `D` dark
- Uppercase = dark/base, lowercase = light/highlight
- Light source: top-left (consistent)

### Character Composition

| Canvas   | Head   | Body ratio | Border |
|---------|--------|-----------|--------|
| 8×8     | 4×4    | 1:1       | 0–1px  |
| 12×12   | 5×5    | 1:1.2     | 1px    |
| 16×16   | 6×6    | 1:1.5     | 1–2px  |
| 24×24   | 8×8    | 1:2       | 2px    |
| 32×32   | 10×10  | 1:2.2     | 2–3px  |
| 48×48   | 14×14  | 1:2.5     | 3–4px  |

- Head proportionally oversized (pixel art convention)
- At least 1px transparent border
- Break perfect symmetry (hair, weapon, pose)
- At 24×24: 2-3 recognizable details > anatomical accuracy (visor line, weapon silhouette, distinct hair)

### Drawing Order
1. Silhouette first → get shape right
2. Base colors → one flat color per material
3. Shadows → darker on bottom-right
4. Highlights → lighter on top-left
5. Details → eyes, accessories, weapon
6. Readability check → zoom out, is shape recognizable?

### Animation Implementation

| Effect   | Implementation                                             |
|---------|-------------------------------------------------------------|
| breathe | Container `translateY(0→-2px)`, 4s infinite                 |
| gleam   | Per-pixel `brightness(1→3→1)`, sweep top→bottom, 5s        |
| wind    | `sin(time*2 + x*0.5)` brightness on hair/cloth             |
| fade-in | Per-pixel `opacity 0→1`, delay `x*0.02 + y*0.04`s         |
| blink   | Eye pixels opacity toggle, 4s cycle                         |
| glow    | `box-shadow` pulse on emissive pixels                       |

---

## Pixel Font Reference (MUST USE for text/numbers)

When drawing text, numbers, or symbols, do NOT guess pixel placement. Use these exact patterns.

### 3×5 Digits (0-9)

```
0: ###  1: .#.  2: ###  3: ###  4: #.#  5: ###  6: ###  7: ###  8: ###  9: ###
   #.#     ##.     ..#     ..#     #.#     #..     #..     ..#     #.#     #.#
   #.#     .#.     ###     .##     ###     ###     ###     .#.     ###     ###
   #.#     .#.     #..     ..#     ..#     ..#     #.#     .#.     #.#     ..#
   ###     ###     ###     ###     ..#     ###     ###     .#.     ###     ###
```

### 3×5 Letters (A-Z)

```
A: .#.  B: ##.  C: ###  D: ##.  E: ###  F: ###  G: ###  H: #.#  I: ###  J: .##
   #.#     #.#     #..     #.#     #..     #..     #..     #.#     .#.     ..#
   ###     ##.     #..     #.#     ###     ##.     #.#     ###     .#.     ..#
   #.#     #.#     #..     #.#     #..     #..     #.#     #.#     .#.     #.#
   #.#     ##.     ###     ##.     ###     #..     ###     #.#     ###     ##.

K: #.#  L: #..  M: #.#  N: #.#  O: ###  P: ###  Q: ###  R: ##.  S: ###  T: ###
   #.#     #..     ###     ##.     #.#     #.#     #.#     #.#     #..     .#.
   ##.     #..     ###     #.#     #.#     ###     #.#     ##.     ###     .#.
   #.#     #..     #.#     #.#     #.#     #..     #.#     #.#     ..#     .#.
   #.#     ###     #.#     #.#     ###     #..     ##.     #.#     ###     .#.

U: #.#  V: #.#  W: #.#  X: #.#  Y: #.#  Z: ###
   #.#     #.#     #.#     .#.     #.#     ..#
   #.#     #.#     ###     .#.     .#.     .#.
   #.#     .#.     ###     .#.     .#.     #..
   ###     .#.     #.#     #.#     .#.     ###
```

### 5×7 Digits (larger, more readable)

```
0: .###.  1: ..#..  2: .###.  3: .###.  4: #..#.  5: #####  6: .###.  7: #####  8: .###.  9: .###.
   #...#     .##..     #...#     #...#     #..#.     #....     #....     ....#     #...#     #...#
   #...#     ..#..     ....#     ....#     #..#.     #....     #....     ...#.     #...#     #...#
   #...#     ..#..     .###.     ..##.     #####     ####.     ####.     ..#..     .###.     .####
   #...#     ..#..     #....     ....#     ...#.     ....#     #...#     ..#..     #...#     ....#
   #...#     ..#..     #....     #...#     ...#.     #...#     #...#     .#...     #...#     #...#
   .###.     .###.     #####     .###.     ...#.     .###.     .###.     .#...     .###.     .###.
```

### Usage rules for text in pixel art:
1. **Always copy from the reference above** — never approximate
2. Use 3×5 for small art (≤16px canvas), 5×7 for larger (≥24px)
3. Leave 1px gap between characters
4. Align to pixel grid — no half-pixel offsets

---

## Core Utility API (for Canvas-based rendering)

Standard helper functions used across all procedural generation. When writing Canvas-based pixel art, include these utilities:

### Deterministic Hash

```javascript
function H(x, y) {
  let n = ((x|0)*374761393 + (y|0)*668265263 + 1013904223)|0;
  n = (((n>>13)^n)*1274126177)|0;
  return (((n>>16)^n)>>>0) / 4294967296;
}
```

Same (x,y) always produces same value 0–1. Foundation for all procedural textures. Enables reproducible, tileable results without seed management.

### Color Manipulation

```javascript
function hx(s) { return [parseInt(s.slice(1,3),16), parseInt(s.slice(3,5),16), parseInt(s.slice(5,7),16)]; }
function dk(c, a) { return c.map(v => Math.max(0, (v*(1-a))|0)); }           // darken
function lt(c, a) { return c.map(v => Math.min(255, (v+(255-v)*a)|0)); }      // lighten
function nz(c, x, y, r) {                                                      // noise inject
  const n = (H(x*7, y*13) - 0.5) * r;
  return c.map(v => Math.max(0, Math.min(255, (v+n)|0)));
}
function pk(pal, x, y) { return [...pal[Math.floor(H(x,y)*pal.length) % pal.length]]; }  // palette pick
function mix(a, b, t) { return a.map((v,i) => Math.round(v + (b[i]-v) * t)); }            // color mix
```

### Noise Range Guidelines

Different materials need different noise amounts for best results:

| Material          | `nz()` range | Why |
|-------------------|-------------|-----|
| Stone, walls      | 8–10        | High variance, natural appearance |
| Wood, planks      | 6–8         | Moderate grain texture |
| Cloth, fabric     | 6–8         | Moderate fold variation |
| Metal, polished   | 4–5         | Subtle, preserves shine |
| Neon, electronics | 15–20       | High flicker, suggests energy |
| Foliage, leaves   | 10–12       | Organic irregularity |
| Sand, dirt        | 4–6         | Gentle speckle |
| Water             | 4–5         | Subtle wave variation |

### PixelBuffer Class

```javascript
class PB {
  constructor(w, h) {
    this.w = w; this.h = h;
    this.rgb = new Uint8ClampedArray(w * h * 3);
    this.mask = new Uint8Array(w * h);
  }
  set(x, y, r, g, b) {
    x |= 0; y |= 0;
    if (x < 0 || x >= this.w || y < 0 || y >= this.h) return;
    const i = (y * this.w + x) * 3;
    this.rgb[i] = r; this.rgb[i+1] = g; this.rgb[i+2] = b;
    this.mask[y * this.w + x] = 1;
  }
  setC(x, y, c) { this.set(x, y, c[0], c[1], c[2]); }
  get(x, y) {
    x |= 0; y |= 0;
    if (x < 0 || x >= this.w || y < 0 || y >= this.h) return null;
    if (!this.mask[y * this.w + x]) return null;
    const i = (y * this.w + x) * 3;
    return [this.rgb[i], this.rgb[i+1], this.rgb[i+2]];
  }
  has(x, y) {
    x |= 0; y |= 0;
    return x >= 0 && x < this.w && y >= 0 && y < this.h && this.mask[y * this.w + x] === 1;
  }
}
```

### Basic Drawing Helpers

```javascript
function fillRect(pb, x1, y1, x2, y2, c) {
  for (let y = y1; y < y2; y++) for (let x = x1; x < x2; x++) pb.setC(x, y, c);
}
function outline(pb, x1, y1, x2, y2, c) {
  for (let x = x1; x < x2; x++) { pb.setC(x, y1, c); pb.setC(x, y2-1, c); }
  for (let y = y1; y < y2; y++) { pb.setC(x1, y, c); pb.setC(x2-1, y, c); }
}
```

---

## Reference Palette Library

Production-validated material palettes (tested across 3000+ assets). Use these as starting points — each palette is an array of RGB values from dark to light:

### Natural Materials

```javascript
const PAL = {
  // Stone & Rock
  stone:     ['#4e5060','#5a6670','#6a7680','#8a96a0','#a0acb4'],  // neutral gray stone
  darkStone: ['#30303a','#3a3a44','#44444e','#505058'],             // dungeon, dark walls
  warmStone: ['#6a5a48','#7a6a58','#8a7a68','#9a8a78','#b0a088'],  // sandstone, desert

  // Wood
  wood:      ['#3a2818','#4a3520','#5a4530','#6a5540','#7a6550'],  // standard dark wood
  lightWood: ['#6a4830','#7a5840','#8a6850','#9a7860','#aa8870'],  // birch, light furniture
  darkWood:  ['#2a1808','#3a2010','#4a3018','#5a3820'],            // ebony, dark beams

  // Foliage
  foliageL:  ['#4a8c30','#5aa040','#68b048','#70c050'],            // sunlit leaves (top)
  foliageD:  ['#2a5018','#3a6a20','#4a7a28','#3a5a20'],            // shaded leaves (bottom)
  tropicalL: ['#40a030','#50b040','#60c050','#70d060'],            // tropical bright
  tropicalD: ['#206010','#308020','#409030','#307020'],            // tropical shade

  // Terrain
  grass:     ['#2a5018','#3a6a20','#4a8030','#5a9040','#68a048'],
  darkGrass: ['#1a3a10','#2a4a18','#3a5a20','#4a6a28'],
  dirt:      ['#5a4a30','#6a5a40','#7a6a50','#8a7a60','#9a8a70'],
  sand:      ['#c0a870','#d0b880','#d8c890','#e0d0a0','#e8d8b0'],
  moss:      ['#3a6020','#4a7030','#5a8040','#6a9050'],

  // Water
  water:     ['#2a4a70','#3a5a80','#4a6a90','#5a7aa0','#6a8ab0'],
  deepWater: ['#1a2a50','#2a3a60','#3a4a70','#4a5a80'],
};
```

### Constructed Materials

```javascript
  // Roof styles
  roofRed:   ['#8a2a20','#a83030','#c04040','#d05050'],
  roofBlue:  ['#2a4a6a','#3a5a7a','#4a6a8a','#5a7a9a'],
  roofBrown: ['#5a4028','#6a5038','#7a6048','#8a7058'],
  thatch:    ['#7a6a30','#8a7a38','#9a8a48','#aa9a58','#baa868'],

  // Metal & Mineral
  iron:      ['#3a3a40','#4a4a50','#5a5a64','#6a6a74'],
  gold:      ['#b08020','#c09030','#d0a040','#e0b850'],
  bone:      ['#c0b8a0','#d0c8b0','#d8d0c0','#e0d8cc'],

  // Fabric & Organic
  cloth:     ['#a03030','#c04040','#d05050','#e06060'],           // red fabric
  clothBlue: ['#3050a0','#4060b0','#5070c0','#6080d0'],
  skin:      ['#c09870','#d0a880','#d8b890','#e0c8a0'],
  hay:       ['#7a5a28','#9a7a38','#b89848','#d0b058','#e8c868'],

  // Special
  lava:      ['#c02010','#e04020','#f06030','#ff8040'],
  coral:     ['#d06060','#e08070','#d0a080','#e0b090'],
  flower:    ['#cc3030','#e05050','#f06060','#ff7070'],
  flowerY:   ['#d0a020','#e0b830','#f0d040'],
  flowerB:   ['#4060c0','#5070d0','#6080e0'],
```

### Warm Palette Convention

Always use saturated, warm-shifted colors for wood/leather/gold — avoid desaturated muddy tones:
```
Wood light:  #c88848 (not #8b6040)
Wood medium: #a86830 (not #6b4530)
Wood dark:   #4a2810 (not #3a2018)
Gold:        #d8a838 (not #a08030)
```

---

## Procedural Texture Engine (for game assets & tilesets)

When the prompt implies a **building**, **game asset**, **tileset**, or **creature** at sizes ≥48px, use the procedural texture engine instead of per-pixel string arrays. The LLM declares **regions + materials**, the engine auto-fills textures.

### Architecture

```
LLM outputs compact declarations:
  → Shape regions (ellipse, rect, triangle composites)
  → Material assignment per region
  → Engine auto-fills procedural textures + shading + outline
```

### Available Procedural Textures (23 types)

| Texture | Algorithm | Use for |
|---------|-----------|---------|
| `texStone` | Voronoi cells + mortar joints | Stone walls, cobblestone, rock |
| `texTiles` | Offset grid + overlap shading | Roof tiles, floor tiles |
| `texRoof` | Offset grid + alternating row offset | Roof tiles with staggered rows |
| `texStucco` | Noisy flat fill | Plaster, stucco, painted walls |
| `texWood` | Sin-wave grain pattern, directional (v/h) | Wood beams, planks, furniture |
| `texPlanks` | Horizontal boards + gap lines + knot spots | Plank floors, chest body, doors |
| `texBark` | Multi-frequency ridges + cracks | Tree trunks, branches |
| `texFoliage` | Elliptical blob + density mask + dual-palette | Bushes, tree canopy, hedges |
| `texWater` | Sinusoidal wave pattern | Rivers, ponds, ocean, puddles |
| `texSand` | Random speckle + gentle noise | Beaches, desert, sandy paths |
| `texMoss` | Sparse overlay with coverage control | Moss on rocks, ruins, damp walls |
| `texFur` | Directional strand pattern | Animal fur, hair, grass |
| `texScales` | Overlapping crescent grid | Fish, reptile, dragon scales |
| `texFeathers` | V-shaft + barb layering | Bird plumage, owl, wings |
| `texSpiral` | Logarithmic spiral + dome shading | Snail shell, ammonite |
| `texSmooth` | Gradient fill with noise | Smooth skin, fins, body |
| `texCloth` | Sinusoidal fold bands + micro-folds | Tents, sacks, banners, draped cloth |
| `texWoven` | Warp/weft grid + geometric motifs | Tent stripes, rugs, baskets, textiles |
| `texMetal` | Specular highlight band + ambient gradient | Iron bands, weapons, shields, armor |
| `texCrystal` | Faceted sectors + internal glow | Gems, crystals, magic items, glass |
| `texBone` | Longitudinal cracks + dry gradient | Skeletons, tusks, horns, fossils |
| `texRope` | Twisted strand pattern + round cross-section | Nets, ropes, bindings, well ropes |
| `texCoral` | Recursive branching growth | Coral, dark tendrils, root structures, vines |

### Core Texture Implementations

**Voronoi Stone** — the most-used texture:
```javascript
function texStone(pb, x1, y1, x2, y2, pal, cellSize, mortarWidth, mortarColor) {
  mortarColor = mortarColor || [30,32,38];
  // Generate random cell centers on a grid
  // For each pixel: find nearest/second-nearest cell center (Manhattan distance)
  // Edge distance (d2-d1) < mortarWidth → mortar color
  // Otherwise → palette color from cell + noise injection
}
```

**Wood Grain** — directional:
```javascript
function texWood(pb, x1, y1, x2, y2, pal, direction) {
  // direction: "v" (vertical grain) or "h" (horizontal)
  // Sin-wave grain: sin(t * 12 + hash-offset) for organic feel
  // Map wave value to palette index
}
```

**Dual-Palette Foliage** — light top, dark bottom:
```javascript
function texFoliage(pb, cx, cy, rx, ry, palLight, palDark, density) {
  // Ellipse mask: dx²/rx² + dy²/ry² ≤ 1
  // Density filter: H(x*3, y*7) < density
  // Top (dy < -0.2) → pick from palLight
  // Bottom → pick from palDark
  // Noise injection for organic feel
}
```

**Roof Tiles** — staggered offset:
```javascript
function texRoof(pb, x1, y1, x2, y2, pal, tileW, tileH) {
  // Offset: odd rows shift by tileW/2
  // Edge pixels (tx==0 or ty==0) → darkened mortar
  // Interior → palette pick with noise
}
```

**Water** — wave pattern:
```javascript
function texWater(pb, x1, y1, x2, y2, pal) {
  // wave = sin((x + y*0.5) * 0.4) * 0.5 + 0.5
  // Map wave to palette index + subtle noise
}
```

### Post-Processing: `pp(pb, shadow, outline)`

Production-validated simple two-parameter post-processing. Returns a new PB (non-destructive):

```javascript
function pp(pb, sh, ol) {
  const out = new PB(pb.w, pb.h);
  for (let y = 0; y < pb.h; y++)
    for (let x = 0; x < pb.w; x++) {
      if (!pb.has(x, y)) continue;
      let c = pb.get(x, y);
      // Directional light: darken toward bottom-right
      const light = 1 - (x/pb.w * 0.15 + y/pb.h * 0.2) * sh * 3;
      c = c.map(v => Math.max(0, Math.min(255, (v * light)|0)));
      // Edge darkening: darken pixels adjacent to empty space
      const edges = [!pb.has(x-1,y), !pb.has(x+1,y), !pb.has(x,y-1), !pb.has(x,y+1)];
      const ec = edges.filter(Boolean).length;
      if (ec > 0) c = dk(c, ol * ec * 0.4);
      out.setC(x, y, c);
    }
  return out;
}
```

**Typical values**: `pp(pb, 0.5, 0.3)` — moderate shadow + outline. Use `pp(pb, 0.8, 0.4)` for dramatic lighting, `pp(pb, 0.3, 0.2)` for subtle/flat items.

For assets needing more control, use the extended multi-pass pipeline:
```
pp(pb, { ao: 0.12, light: 0.1, specular: [{cx,cy,r,intensity}], outline: 0.28 })
```
- **passAO** — ambient occlusion at region boundaries
- **passLight** — directional light from top-left
- **passSpecular** — highlights for metal, crystal, glass
- **passOutline** — edge darkening

### Composition System

**RGBA PixelBuffer** — All pixels have alpha channel. Supports semi-transparent effects (glow halos, glass, cloth veils).

**`blitAlpha(src, dst, dx, dy, opts)`** — Alpha-aware blit with contact shadow:
```javascript
blitAlpha(sack, scene, 10, 20, {
  contactShadow: true,      // auto-generate shadow below source
  shadowOffset: [1, 2],      // shadow displacement
  shadowAlpha: 0.18,          // shadow opacity
  alpha: 1.0                  // global source opacity
});
```

**`blit(src, dst, dx, dy)`** — Simple copy for tileset assembly:
```javascript
function blit(src, dst, dx, dy) {
  for (let y = 0; y < src.h; y++)
    for (let x = 0; x < src.w; x++) {
      if (!src.has(x, y)) continue;
      dst.setC(dx + x, dy + y, src.get(x, y));
    }
}
```

**Assembly pattern** — Build complex assets from layered sub-parts:
```
Assembly = base_structure + blitAlpha(items, contactShadow) + decoration
Example: fruit_stall = wood_frame + cloth_awning(texCloth) + blitAlpha(fruit_crate) × 4
```

### Shape Primitives

| Primitive | Usage |
|-----------|-------|
| `inE(x,y, cx,cy, rx,ry)` | Ellipse — body parts, rocks, barrels, foliage |
| `inTri(x,y, x1,y1, x2,y2, x3,y3)` | Triangle — roofs, crystal shards |
| `inPoly(x,y, vertices[])` | Arbitrary polygon — complex silhouettes |
| `inRR(x,y, x1,y1, x2,y2, r)` | Rounded rectangle — crates, signs, shields |
| `distY(y, x, freq, amp)` | Sinusoidal Y warp — cloth edges, organic forms |

**Ellipse masking** — the most-used shape technique (15+ uses per pack):
```javascript
const dx = (x - cx) / rx, dy = (y - cy) / ry;
if (dx * dx + dy * dy > 1) continue;  // outside ellipse
```
Use for: tree canopies, character heads, shields, gems, barrel tops, cloud shapes. Avoids explicit alpha — shapes defined by mask test.

### Building Construction Order

When generating buildings/structures, always layer in this order:

1. **Walls** — `texStone` / `texWood` / `texStucco` for the main body
2. **Roof** — `texRoof` / `texTiles` overlaid on top, often with triangle geometry
3. **Openings** — `fillRect` for door/window holes, then fill with appropriate material
4. **Frames** — `texWood` for door/window frames, `outline()` for borders
5. **Details** — chimney, signs, decorative elements, potted plants
6. **Post-processing** — `pp()` for directional light + edge outline

### Per-Material Shading Guidance

| Material | Tones | Shading pattern |
|----------|-------|----------------|
| Stone | 5 | deep mortar → stone shadow → base → light → highlight chip |
| Wood | 4 | grain dark → grain base → grain light → edge highlight |
| Cloth | 5 | fold valley → fold shadow → base → fold light → fold peak |
| Metal | 6 | dark ambient → base → reflected mid → reflected light → specular edge → specular peak |
| Crystal | 5 | core glow → base facet → facet edge → bright facet → specular point |
| Bone | 4 | crack shadow → aged base → surface light → dry highlight |

### Dual-Palette Shading Strategy

Instead of per-pixel lighting calculations, use **two palettes** (light + dark) for each material:

```javascript
// Define light and dark variants
const folLight = ['#4a8c30','#5aa040','#68b048','#70c050'];
const folDark  = ['#2a5018','#3a6a20','#4a7a28','#3a5a20'];

// In rendering: pick palette based on position
const isTop = dy < -0.2;  // top = lit, bottom = shadowed
const c = isTop ? pk(folLight, x, y) : pk(folDark, x, y);
```

This is simpler and faster than per-pixel shading. Works especially well for foliage, character clothing, and large surfaces.

### Neon Glow Layering (for cyberpunk/magic themes)

Three-layer technique for glowing elements:

```javascript
// Layer 1: Base element (full brightness)
pb.setC(x, y, nz(pk(neonGreen, x, y), x, y, 5));
// Layer 2: Glow halo (darker, spread wider, more noise)
pb.setC(x, y-1, nz(dk(neonGreen[2], 0.6), x, y-1, 8));
// Layer 3: Hot-spot highlight (over-bright center)
pb.setC(x, y, lt(baseColor, 0.3));
```

For wet surface reflections:
```javascript
const ref = mix(baseColor, neonColor, 0.15);  // subtle tint, not full color
pb.setC(x, y, nz(ref, x, y, 5));
```

### Asset Size Minimums

| Asset type | Minimum size | Recommended |
|-----------|-------------|-------------|
| Hero building | 64px | 80–120px |
| Furniture / structures | 32px | 40–56px |
| Containers (crates, barrels) | 20px | 24–32px |
| Small items (potions, food) | 12px | 16–20px |
| Detail items (pebbles, coins) | 8px | 8–12px |
| Characters | 18px | 24–36px |

### Rendering

Uses `<canvas>` with `PixelBuffer` class. Each pixel is individually addressable. Rendered at configurable scale (3–6x typical) with `image-rendering: pixelated`.

---

## Sprite Template System (for items ≤20px)

For small items (potions, books, bowls, weapons, etc.), do NOT use procedural texture fills. Use **character-grid templates** with material zone annotations. The engine auto-shades each zone based on material type.

### Template Format

```javascript
const POTION = {
  grid: [
    '..cc..',
    '..kk..',
    '.GGGG.',
    'GLLLLG',
    'GLLLLG',
    'GLLLLG',
    'GLLLLG',
    '.GGGG.',
  ],
  materials: { c:'cork', k:'cork', G:{type:'glass',alpha:200}, L:'liquid' }
};
```

LLM hand-crafts the **shape** (which pixels belong to which zone). Engine handles all **shading** automatically per material type.

### Material Shader Types

Each material zone gets its own shading strategy:

| Shader | Effect | Use for |
|--------|--------|---------|
| `wood` | Directional warm light, grain hint | Handles, frames, furniture |
| `metal` | Sharp specular band + cool ambient | Blades, armor, iron bands |
| `gold` | Warm specular + rich ambient | Crowns, coins, fittings |
| `glass` | Cylindrical highlight + transparency | Bottles, windows, lenses |
| `liquid` | Horizontal gradient + surface line | Potions, water, soup |
| `cloth` | Soft folds + directional | Fabric, sacks, pillows |
| `leather` | Warm directional, moderate contrast | Covers, straps, seats |
| `ceramic` | Spherical highlight | Bowls, vases, pots |
| `gem` | Facet shimmer + top-left specular | Gems, crystals |
| `paper` | Subtle vertical gradient | Scrolls, books interior |
| `cork` | Warm top-light | Stoppers, small wood |
| `fire` | Strong top-bright, very warm | Flames, embers |
| `plant` | Cool-warm directional | Leaves, stems |
| `flat` | No shading | UI elements, symbols |

### Hue-Shift Shading

Replace simple darken/lighten with hue-aware shade function:
```javascript
shade(color, amount, warmth)
// amount: -1 (shadow) to +1 (highlight)
// warmth: 0 (neutral) to 1 (warm highlights, cool shadows)
// Highlights shift toward yellow/white, shadows shift toward blue/purple
```

### Adaptive Outline

Outline color = saturated dark version of the adjacent pixel color (not uniform darkening):
- Wood edge → deep brown (not gray)
- Metal edge → dark blue-gray (not dark gray)
- Plant edge → deep green (not muddy green)

---

## Tileset Mode

When prompt implies **tileset / sprite sheet / asset pack**, generate a complete sheet of related game-ready assets on a single canvas.

### Architecture

```
1. Shared engine (PB, hash, color utils, geometry, all texture generators, post-processing)
2. Shared palette (all assets use the same color sets for consistency)
3. Asset factories — each returns a standalone PB:
     function mkCottage() { const p = new PB(80,100); /* layers */ return pp(p); }
     function mkBush(rx,ry) { const p = new PB(...); texFoliage(...); return pp(p); }
     function mkRock(w,h) { ... }
4. Composition — blit all assets onto a master canvas:
     const sheet = new PB(370, 280);
     blit(mkCottage(), sheet, 2, 4);
     blit(mkBush(13,11), sheet, 160, 4);
5. Render master canvas at 3x scale
```

### Tileset Composition Rules

1. **Layout**: Place the largest/hero asset (building, character) in the top-left. Group related smaller assets by category across the sheet.
2. **Scale**: Use 3x rendering scale (gives clear pixels without being too large). Sheet canvas is typically 340–400 × 250–300 pixels.
3. **Background**: Neutral gray `#c0c0b8` fills the sheet — standard tileset convention.
4. **Shared palette**: Define ALL color palettes upfront as a shared `P` object. Every factory reads from `P`. This guarantees style consistency.
5. **Post-processing**: Apply `pp()` (autoShade + edgeOutline) to each individual asset PB, NOT to the whole sheet. This prevents cross-asset shading artifacts.
6. **Blit helper**: Use `blit(src, dst, dx, dy)` to copy asset PBs onto the sheet at specific positions.

### Asset Factory Pattern

Each asset type is a function that returns a self-contained `PB`:

```javascript
// Configurable factory — same function, different params → different variants
function mkBush(w, h, rx, ry) {
  const p = new PB(w, h);
  texFoliage(p, w/2, h/2, rx, ry, P.fol, P.folDk, .82);
  return pp(p, 0.5, 0.3);  // auto-shade + outline
}

// Variant factory — type parameter controls shape
function mkFence(type) {  // 'panel' | 'gate' | 'post'
  const p = new PB(22, 34);
  // posts, beams, iron bands based on type
  return pp(p, 0.5, 0.3);
}

// Call with different params to create variants:
blit(mkBush(30,26,13,11), sheet, 160, 4);   // large
blit(mkBush(24,20,10,8), sheet, 196, 8);     // medium
blit(mkBush(14,12,5,4), sheet, 316, 10);     // small
```

### Theme → Asset List

When generating a tileset, auto-decide which assets to include based on theme:

| Theme | Hero asset | Structures | Nature | Terrain | Details |
|-------|-----------|------------|--------|---------|---------|
| **Medieval village** | Stone cottage | Fences, crates, well, birdhouse | Bushes, trees, flowers | Cobblestone paths, rocks, cliffs | Grass, stumps, pebbles |
| **Castle** | Keep / throne room | Marble floors, arches, banners, pillars | Courtyard garden | Stone floor tiles, walls | Torches, candelabra, shields |
| **Market / Bazaar** | Market tent (texCloth awning) | Stalls with goods, weapon rack, potion cart | Potted plants, flowers | Cobblestone, carpet (texWoven) | Fruits, bottles, coins, banners |
| **Fishing village** | Wooden warehouse | Shelf racks, barrels, nets, drying rack | Seaweed, coral | Dock planks, water tiles | Fish, crates, sacks, rope, anchor |
| **Dungeon** | Castle gate | Torches, doors, chests, barrels | Moss, vines, mushrooms | Stone floor tiles, walls, pits | Bones (texBone), chains, potions |
| **Dark fantasy** | Corrupted tower | Spider structures, bone altars, cursed trees | Crystal growths (texCrystal), dark roots (texCoral) | Cracked stone, void tiles | Webs, candles, skull, runes |
| **Tropical / Tribal** | Totem pillar, tiki hut | Yurt/tent (texWoven), palm shelter | Palm trees, coral (texCoral), shells | Sand, beach tiles, shallow water | Tiki masks, seashells, rope (texRope) |
| **Forest** | Large oak tree | Campfire, tent (texCloth), log cabin | Bushes, ferns, saplings | Dirt paths, streams, boulders | Mushrooms, berries, acorns |
| **Farm** | Barn | Fences, troughs, windmill | Crops, haystacks, flowers | Dirt/grass tiles, pond | Chickens, tools, baskets |
| **Desert** | Pyramid / obelisk | Sand-brick walls, clay pots, market tent | Cacti, palm trees, dry shrubs | Sand tiles, cracked earth | Scarab, bones, gems, urns |
| **Snow mountain** | Lodge / cabin | Fences, sleds, ice blocks | Snow-capped pines, frozen bushes | Snow tiles, ice, frozen paths | Icicles, footprints, snowman |
| **Cyberpunk** | Neon building facade | Vending machines, hologram signs, dumpsters | None (urban) | Asphalt, grating, wet road | Neon signs, cables, steam vents |
| **Japanese garden** | Pagoda / shrine | Torii gate, stone lantern, wooden bridge | Cherry trees, bamboo, bonsai | Stone path, koi pond, gravel | Koi fish, lotus, paper lantern |
| **Pirate** | Ship / dock | Barrels, cannons, treasure chest, tavern sign | Palm trees, seaweed | Dock planks, water, sand | Flags, parrots, rum bottles, maps |
| **Space station** | Airlock / control panel | Terminals, cryo-pods, supply crates | Alien flora | Metal floor tiles, pipes, grating | Monitors, wires, oxygen tanks |

**Assembly recipes** for composite assets:
```
market_stall = wood_frame + texCloth(awning, stripes) + blitAlpha(fruit_crate×4) + contact_shadows
weapon_rack = wood_frame + blitAlpha(sword, texMetal) + blitAlpha(axe) + blitAlpha(shield)
shelf_unit = wood_shelves + blitAlpha(sack, texCloth) + blitAlpha(barrel) + blitAlpha(crate)
fishing_net = rope_frame(texRope) + blitAlpha(hanging_fish×4)
cottage = texStone(walls) + texRoof(roof, trapezoid) + texWood(door_frame) + fillRect(windows) + pp()
```

Aim for **25–40 assets** per tileset, covering all categories.

---

### Atmosphere Implementation

| Element | CSS/JS                                                     |
|---------|-------------------------------------------------------------|
| stars   | Random 1-2px dots, opacity twinkle                          |
| moon    | `border-radius:50%` + radial-gradient + box-shadow glow    |
| rain    | 1px divs, fast vertical translateY                          |
| petals  | `border-radius:50% 0 50% 0`, rotate + fall                 |
| mist    | Translucent gradient div, slow horizontal drift             |
| embers  | Small dots rising with box-shadow glow                      |
| snow    | Small circles, slow fall + horizontal sway                  |
| clouds  | Multi-layer semi-transparent rounded rects + slow translate  |
| fireflies | Small dots with glow, random walk paths                   |
