# PixelArt Skill

Generate pixel art as self-contained HTML. No images, no dependencies.

## Interface

### Mode 1: Single asset (default)

**Input:** size + prompt.

```
/pixelart 16x16, a fire mage
/pixelart 32x32, cyberpunk cat with neon goggles
/pixelart 80x85, stone cottage with red tile roof
/pixelart 50x40, a fox sitting in grass
```

### Mode 2: Tileset / Sprite sheet

**Input:** `tileset` + theme. Generates 25–40 related assets on a single sheet.

```
/pixelart tileset, medieval village
/pixelart tileset, dungeon crawler
/pixelart tileset, forest nature pack
/pixelart tileset, farm animals
```

**Output:** A single, self-contained `.html` file.

---

## How to decide everything else

You only receive **size** and **prompt**. Use the rules below to decide detail level, palette, rendering method, animations, and atmosphere automatically.

### Step 1: Size → Detail Level

| Size             | Detail  | Max Colors | Shading                                   |
|-----------------|---------|-----------|-------------------------------------------|
| 8×8             | minimal | 3–5       | Flat colors, no shading                    |
| 12×12           | standard| 5–8       | 1 shadow tone per material                 |
| 16×16           | standard| 8–12      | 1 shadow tone per material                 |
| 16×24, 24×24    | detailed| 10–16     | Highlight + base + shadow per material     |
| 32×32+          | detailed| 12–20     | Highlight + base + shadow, texture hints   |
| 48×48+          | ultra   | 15–25     | Multi-layer shading, dithering gradients   |
| 96×64+ (canvas) | ultra   | Unlimited | Procedural, FBM noise, Bayer dithering     |
| 48×96+ (asset)  | ultra   | Unlimited | Procedural texture engine, region-based    |

### Step 2: Prompt → Rendering Method

Read the prompt and pick:

| Prompt implies...              | Render method                          |
|-------------------------------|----------------------------------------|
| Simple icon / item / emoji     | **CSS box-shadow** (zero JS, static)   |
| Character / sprite             | **CSS Grid + JS** (per-pixel control)  |
| "animated", movement, effects  | **Grid + animation JS**                |
| Landscape, scene, environment  | **Canvas** (procedural landscape)      |
| Building, tileset, game asset  | **Canvas** (procedural texture engine) |
| Animal, creature, organic      | **Canvas** (procedural organic textures)|

### Step 3: Prompt → Animations & Atmosphere

Auto-add based on content:

| Content in prompt        | Auto-add                                       |
|-------------------------|-------------------------------------------------|
| Character with weapon    | `breathe` + `gleam` (weapon shine)              |
| Character with hair/cloth| `breathe` + `wind` (hair/cloth flutter)         |
| Has eyes                 | `blink` (periodic)                              |
| Night / dark scene       | Stars, moon, mist                               |
| Fire / lava              | Ember particles, glow                           |
| Water / lake / ocean     | Water reflection animation                      |
| Rain / storm             | Rain particles                                  |
| Cherry blossom / spring  | Falling petals                                  |
| Snow / winter            | Snowflake particles                             |
| Magic / glow / neon      | Glow pulse on emissive pixels                   |
| City / urban             | Building silhouettes, lit windows               |
| Any character            | `fade-in` (staggered pixel reveal on load)      |

If the prompt doesn't suggest any effects, output static art with no animation.

---

## Output Format

### Small art (≤48×48): String-array data + HTML renderer

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>{{name}} · Pixel Art</title>
<style>
  /* Dark background, centered display, animation keyframes */
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
    // Each single character → one hex color
    // Use semantic letters: S=skin, H=hair, E=eyes, B=body, etc.
  },
  pixels: [
    // Each string = one row, each char = one pixel
    // '.' = transparent, letter = palette color
  ]
};

// Grid renderer + animations
</script>
</body>
</html>
```

### Large art (≥64px wide): Canvas-based procedural generation

Use `<canvas>` with:
- **FBM noise** (fractal Brownian motion) for terrain, clouds, water
- **Ridge noise** for mountain ridgelines
- **Bayer 4×4 ordered dithering** for palette-limited gradients
- **Layered rendering**: sky → celestial → clouds → mountains → trees → ground → water → mist
- **PixelBuffer** class for pixel-level manipulation + alpha blending

```javascript
// Noise engine
function hash2D(x, y) { /* integer hash → 0-1 */ }
function valueNoise(x, y) { /* smoothstep interpolated */ }
function fbm(x, y, octaves) { /* layered noise */ }

// Bayer dithering
const BAYER = [[0,8,2,10],[12,4,14,6],[3,11,1,9],[15,7,13,5]];
function ditherColor(x, y, c1, c2, ratio) { /* ordered dither between two colors */ }

// PixelBuffer with set, get, blend, tri, rect, line
// Layer functions: drawSky, drawMountains, drawTrees, drawWater, drawBuildings, etc.
```

---

## Design Rules (always follow these)

### Palette

- `.` is always transparent
- Use semantic character mapping: `S` skin, `H` hair, `E` eyes, `B` body, `M` mouth, `A` armor/metal, `G` gold, `R` red, `W` white, `D` dark
- Uppercase = dark/base, lowercase = light/highlight (convention)
- Every material: at minimum a base color; at detailed level add shadow + highlight
- Light source: top-left (consistent across all materials)

### Composition

| Canvas        | Head    | Body ratio | Border padding |
|--------------|---------|-----------|----------------|
| 8×8          | 4×4     | 1:1       | 0–1px          |
| 12×12        | 5×5     | 1:1.2     | 1px            |
| 16×16        | 6×6     | 1:1.5     | 1–2px          |
| 24×24        | 8×8     | 1:2       | 2px            |
| 32×32        | 10×10   | 1:2.2     | 2–3px          |
| 48×48        | 14×14   | 1:2.5     | 3–4px          |

- Head is proportionally oversized (pixel art convention)
- Leave transparent border for breathing room
- Break perfect symmetry with details (hair, weapon, pose)
- At 24×24: 2-3 recognizable details > anatomical accuracy (visor line, weapon silhouette, distinct hair)

### Pixel Font Reference (MUST USE for text/numbers)

When drawing text, numbers, or symbols, **always copy from these patterns**. Never guess.

**3×5 Digits:**
```
0: ###  1: .#.  2: ###  3: ###  4: #.#  5: ###  6: ###  7: ###  8: ###  9: ###
   #.#     ##.     ..#     ..#     #.#     #..     #..     ..#     #.#     #.#
   #.#     .#.     ###     .##     ###     ###     ###     .#.     ###     ###
   #.#     .#.     #..     ..#     ..#     ..#     #.#     .#.     #.#     ..#
   ###     ###     ###     ###     ..#     ###     ###     .#.     ###     ###
```

**5×7 Digits (for ≥24px canvas):**
```
0: .###.  1: ..#..  2: .###.  3: .###.  4: #..#.  5: #####  6: .###.  7: #####  8: .###.  9: .###.
   #...#     .##..     #...#     #...#     #..#.     #....     #....     ....#     #...#     #...#
   #...#     ..#..     ....#     ....#     #..#.     #....     #....     ...#.     #...#     #...#
   #...#     ..#..     .###.     ..##.     #####     ####.     ####.     ..#..     .###.     .####
   #...#     ..#..     #....     ....#     ...#.     ....#     #...#     ..#..     #...#     ....#
   #...#     ..#..     #....     #...#     ...#.     #...#     #...#     .#...     #...#     #...#
   .###.     .###.     #####     .###.     ...#.     .###.     .###.     .#...     .###.     .###.
```

Use 3×5 for small canvas (≤16px), 5×7 for larger. Always 1px gap between characters.

### Drawing Process

1. Start with the **silhouette** — get the shape right first
2. Fill in **base colors** — one flat color per material
3. Add **shadows** — darker tone on bottom-right edges
4. Add **highlights** — lighter tone on top-left edges
5. Add **details** — eyes, buttons, weapon, accessories
6. Check readability — zoom out, is the shape recognizable?

### Animation Implementation

| Effect    | CSS/JS                                                      |
|----------|-------------------------------------------------------------|
| breathe  | Container `translateY(0→-2px)`, 4s ease-in-out infinite     |
| gleam    | Per-pixel `brightness(1→3→1)`, sweep top→bottom, 5s interval|
| wind     | `sin(time*2 + x*0.5)` → brightness filter on hair/cloth    |
| fade-in  | Per-pixel `opacity 0→1`, delay = `x*0.02 + y*0.04`s       |
| blink    | Eye pixels opacity toggle, 4s cycle                         |
| glow     | `box-shadow` pulse on emissive pixels                       |

### Atmosphere Implementation

| Element  | Pure CSS/JS                                                 |
|---------|-------------------------------------------------------------|
| stars   | Random 1-2px dots, `opacity` twinkle animation              |
| moon    | `border-radius:50%` + `radial-gradient` + `box-shadow` glow|
| rain    | 1px wide divs, fast vertical `translateY` animation         |
| petals  | `border-radius:50% 0 50% 0`, rotate + fall animation       |
| mist    | Translucent gradient div, slow horizontal drift             |
| embers  | Small dots rising with `box-shadow` glow                    |
| snow    | Small circles, slow fall + horizontal sway                  |
| clouds  | Multi-layer semi-transparent rounded rects + slow translate  |
| fireflies | Small dots with glow, random walk paths                   |

---

## Core Utility API (for Canvas-based rendering)

Standard helper functions used across all procedural generation. When writing Canvas-based pixel art, include these utilities.

### Deterministic Hash

```javascript
function H(x, y) {
  let n = ((x|0)*374761393 + (y|0)*668265263 + 1013904223)|0;
  n = (((n>>13)^n)*1274126177)|0;
  return (((n>>16)^n)>>>0) / 4294967296;
}
```

Same (x,y) always produces same value 0–1. Foundation for all procedural textures. Enables reproducible, tileable results.

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

Production-validated material palettes (tested across 3000+ assets). Use these as starting points — each palette is an array of hex colors from dark to light:

### Natural Materials

```javascript
const PAL = {
  stone:     ['#4e5060','#5a6670','#6a7680','#8a96a0','#a0acb4'],  // neutral gray stone
  darkStone: ['#30303a','#3a3a44','#44444e','#505058'],             // dungeon walls
  warmStone: ['#6a5a48','#7a6a58','#8a7a68','#9a8a78','#b0a088'],  // sandstone

  wood:      ['#3a2818','#4a3520','#5a4530','#6a5540','#7a6550'],  // dark wood
  lightWood: ['#6a4830','#7a5840','#8a6850','#9a7860','#aa8870'],  // birch
  darkWood:  ['#2a1808','#3a2010','#4a3018','#5a3820'],            // ebony

  foliageL:  ['#4a8c30','#5aa040','#68b048','#70c050'],            // sunlit leaves
  foliageD:  ['#2a5018','#3a6a20','#4a7a28','#3a5a20'],            // shaded leaves

  grass:     ['#2a5018','#3a6a20','#4a8030','#5a9040','#68a048'],
  dirt:      ['#5a4a30','#6a5a40','#7a6a50','#8a7a60','#9a8a70'],
  sand:      ['#c0a870','#d0b880','#d8c890','#e0d0a0','#e8d8b0'],
  moss:      ['#3a6020','#4a7030','#5a8040','#6a9050'],
  water:     ['#2a4a70','#3a5a80','#4a6a90','#5a7aa0','#6a8ab0'],
  deepWater: ['#1a2a50','#2a3a60','#3a4a70','#4a5a80'],
};
```

### Constructed Materials

```javascript
  roofRed:   ['#8a2a20','#a83030','#c04040','#d05050'],
  roofBlue:  ['#2a4a6a','#3a5a7a','#4a6a8a','#5a7a9a'],
  roofBrown: ['#5a4028','#6a5038','#7a6048','#8a7058'],
  thatch:    ['#7a6a30','#8a7a38','#9a8a48','#aa9a58','#baa868'],

  iron:      ['#3a3a40','#4a4a50','#5a5a64','#6a6a74'],
  gold:      ['#b08020','#c09030','#d0a040','#e0b850'],
  bone:      ['#c0b8a0','#d0c8b0','#d8d0c0','#e0d8cc'],

  cloth:     ['#a03030','#c04040','#d05050','#e06060'],
  clothBlue: ['#3050a0','#4060b0','#5070c0','#6080d0'],
  skin:      ['#c09870','#d0a880','#d8b890','#e0c8a0'],
  hay:       ['#7a5a28','#9a7a38','#b89848','#d0b058','#e8c868'],
  lava:      ['#c02010','#e04020','#f06030','#ff8040'],
  coral:     ['#d06060','#e08070','#d0a080','#e0b090'],
  flower:    ['#cc3030','#e05050','#f06060','#ff7070'],
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

## Procedural Texture Engine (game assets, tilesets, creatures)

When the prompt implies a **building**, **game asset**, **tileset**, or **creature** at sizes ≥48px, use the procedural texture engine. Instead of specifying every pixel, declare **regions + materials** and let the engine fill textures automatically.

### How it works

```
LLM declares (compact):
  { region: "rect", bounds: [12,47,68,74], material: "stone_wall" }
  { region: "triangle", verts: [[40,14],[4,48],[76,48]], material: "roof_tiles" }

Engine auto-fills:
  → Procedural texture per material
  → Auto-shading (directional light)
  → Edge outline detection
  → Local shadows
```

### Procedural Textures (23 types)

| Texture | Algorithm | Use for |
|---------|-----------|---------|
| `texStone` | Voronoi cells + mortar joints | Stone walls, cobblestone, rock surfaces |
| `texTiles` | Offset grid + overlap shading | Roof tiles, floor tiles, paving |
| `texRoof` | Offset grid + alternating row offset | Roof tiles with staggered rows |
| `texStucco` | Noisy flat fill | Plaster, stucco, painted walls |
| `texWood` | Sin-wave grain pattern, directional (v/h) | Wood beams, door, furniture |
| `texPlanks` | Horizontal boards + gap lines + knot spots | Plank floors, chest body, siding |
| `texBark` | Multi-frequency ridges + cracks | Tree trunks, branches |
| `texFoliage` | Elliptical blob + density mask + dual-palette | Bushes, tree canopy, hedges |
| `texWater` | Sinusoidal wave pattern | Rivers, ponds, ocean, puddles |
| `texSand` | Random speckle + gentle noise | Beaches, desert, sandy paths |
| `texMoss` | Sparse overlay with coverage control | Moss on rocks, ruins, damp walls |
| `texFur` | Directional strand pattern | Animal fur, hair |
| `texScales` | Overlapping crescent grid | Fish, reptile, dragon scales |
| `texFeathers` | V-shaft + barb layering | Bird plumage, wings |
| `texSpiral` | Logarithmic spiral + dome shading | Snail shell, decorative spiral |
| `texSmooth` | Gradient fill with noise | Smooth skin, fins, body |
| `texCloth` | Sinusoidal fold bands + micro-folds | Tents, sacks, banners, draped cloth |
| `texWoven` | Warp/weft grid + geometric motifs | Tent stripes, rugs, baskets, textiles |
| `texMetal` | Specular highlight band + ambient gradient | Iron bands, weapons, shields, armor |
| `texCrystal` | Faceted sectors + internal glow | Gems, crystals, magic items, glass |
| `texBone` | Longitudinal cracks + dry gradient | Skeletons, tusks, horns, fossils |
| `texRope` | Twisted strand pattern + round cross-section | Nets, ropes, bindings |
| `texCoral` | Recursive branching growth | Coral, dark tendrils, vines, roots |

### Core Texture Implementations

**Voronoi Stone** — the most-used texture:
```javascript
function texStone(pb, x1, y1, x2, y2, pal, cellSize, mortarWidth, mortarColor) {
  // Generate random cell centers on a grid
  // For each pixel: find nearest/second-nearest cell center (Manhattan distance)
  // Edge distance (d2-d1) < mortarWidth → mortar color
  // Otherwise → palette color from cell + noise injection
}
```

**Dual-Palette Foliage** — light top, dark bottom:
```javascript
function texFoliage(pb, cx, cy, rx, ry, palLight, palDark, density) {
  // Ellipse mask: dx²/rx² + dy²/ry² ≤ 1
  // Density filter: H(x*3, y*7) < density
  // Top (dy < -0.2) → pick from palLight, Bottom → pick from palDark
}
```

### Post-Processing Pipeline

**Simple (production-validated):**
```javascript
pp(pb, shadowAmount, outlineAmount)
// shadowAmount: directional light strength (0.3 subtle → 0.8 dramatic)
// outlineAmount: edge darkening strength (0.2 subtle → 0.4 strong)
// Returns new PB (non-destructive)
```

**Extended multi-pass (for complex assets):**
1. **`autoShade(strength)`** — Global directional light from top-left
2. **`edgeOutline(amount)`** — Darken pixels adjacent to empty space (outline effect)
3. **`shadowRect(...)`** — Local shadow with directional falloff (under eaves, ground shadow)
4. **`addGlow(cx, cy, radius, color, intensity)`** — Radial light emission (windows, magic)

### Shape Definition

Shapes are defined as composites of primitive tests:

- **`inE(x, y, cx, cy, rx, ry)`** — Ellipse (body parts, rocks, foliage blobs)
- **`inTri(x, y, ...verts)`** — Triangle (roofs, gables, ears)
- **Rect** — Simple bounds check (walls, doors, windows)
- **Composite** — Multiple primitives combined for complex organic shapes

**Ellipse masking** — the most-used shape technique:
```javascript
const dx = (x - cx) / rx, dy = (y - cy) / ry;
if (dx * dx + dy * dy > 1) continue;  // outside ellipse
```

### Building Construction Order

When generating buildings/structures, always layer in this order:
1. **Walls** — `texStone` / `texWood` / `texStucco` for the main body
2. **Roof** — `texRoof` / `texTiles` overlaid on top, often with triangle geometry
3. **Openings** — `fillRect` for door/window holes, then fill with appropriate material
4. **Frames** — `texWood` for door/window frames, `outline()` for borders
5. **Details** — chimney, signs, decorative elements, potted plants
6. **Post-processing** — `pp()` for directional light + edge outline

### Per-Material Shading

| Material | Tones | Shading pattern |
|----------|-------|----------------|
| Stone | 5 | deep mortar → stone shadow → base → light → highlight chip |
| Wood | 4 | grain dark → grain base → grain light → edge highlight |
| Cloth | 5 | fold valley → fold shadow → base → fold light → fold peak |
| Metal | 6 | dark ambient → base → reflected mid → reflected light → specular edge → specular peak |
| Crystal | 5 | core glow → base facet → facet edge → bright facet → specular point |
| Bone | 4 | crack shadow → aged base → surface light → dry highlight |

### Dual-Palette Shading Strategy

Instead of per-pixel lighting, use **two palettes** (light + dark) for each material:
```javascript
const folLight = ['#4a8c30','#5aa040','#68b048','#70c050'];
const folDark  = ['#2a5018','#3a6a20','#4a7a28','#3a5a20'];

const isTop = dy < -0.2;  // top = lit, bottom = shadowed
const c = isTop ? pk(folLight, x, y) : pk(folDark, x, y);
```

### Neon Glow Layering (cyberpunk/magic themes)

Three-layer technique:
```javascript
// Layer 1: Base element (full brightness)
pb.setC(x, y, nz(pk(neonGreen, x, y), x, y, 5));
// Layer 2: Glow halo (darker, spread wider)
pb.setC(x, y-1, nz(dk(neonGreen[2], 0.6), x, y-1, 8));
// Layer 3: Hot-spot highlight
pb.setC(x, y, lt(baseColor, 0.3));
```

For wet surface reflections:
```javascript
const ref = mix(baseColor, neonColor, 0.15);  // subtle tint
```

### Composition System

**`blitAlpha(src, dst, dx, dy, opts)`** — Alpha-aware blit with contact shadow:
```javascript
blitAlpha(sack, scene, 10, 20, {
  contactShadow: true, shadowOffset: [1,2], shadowAlpha: 0.18
});
```

**Assembly pattern:**
```
cottage = texStone(walls) + texRoof(roof, trapezoid) + texWood(door_frame) + fillRect(windows) + pp()
market_stall = wood_frame + texCloth(awning) + blitAlpha(fruit_crate×4) + contact_shadows
```

### Asset Sizes

| Asset type | Minimum | Recommended |
|-----------|---------|-------------|
| Hero building | 64px | 80–120px |
| Furniture | 32px | 40–56px |
| Containers | 20px | 24–32px |
| Small items | 12px | 16–20px |
| Detail items | 8px | 8–12px |
| Characters | 18px | 24–36px |

### Single Asset Examples

| Prompt | Engine produces |
|--------|----------------|
| `80x85, stone cottage with red roof` | Voronoi stone wall + offset-grid tiles + stucco gable + arched wood door + glass windows |
| `48x90, wizard tower at night` | Dark stone + purple conical roof + glowing windows with addGlow + flag |
| `50x40, fox sitting in grass` | Fur texture with per-zone direction angles + ellipse composite body |
| `44x26, koi fish in pond` | Scale overlay + gradient fins + water ripple background |

---

## Sprite Template System (for items ≤20px)

For small items (potions, books, bowls, weapons), use **character-grid templates** with material zone annotations:

```javascript
const POTION = {
  grid: [
    '..cc..',
    '..kk..',
    '.GGGG.',
    'GLLLLG',
    'GLLLLG',
    '.GGGG.',
  ],
  materials: { c:'cork', k:'cork', G:{type:'glass',alpha:200}, L:'liquid' }
};
```

### Material Shaders

| Shader | Effect | Use for |
|--------|--------|---------|
| `wood` | Directional warm light, grain hint | Handles, frames |
| `metal` | Sharp specular band + cool ambient | Blades, armor |
| `gold` | Warm specular + rich ambient | Crowns, coins |
| `glass` | Cylindrical highlight + transparency | Bottles, windows |
| `liquid` | Horizontal gradient + surface line | Potions, water |
| `cloth` | Soft folds + directional | Fabric, sacks |
| `leather` | Warm directional | Covers, straps |
| `ceramic` | Spherical highlight | Bowls, vases |
| `gem` | Facet shimmer + specular | Gems, crystals |
| `paper` | Subtle vertical gradient | Scrolls, books |
| `cork` | Warm top-light | Stoppers |
| `fire` | Strong top-bright, warm | Flames, embers |
| `plant` | Cool-warm directional | Leaves, stems |
| `flat` | No shading | UI elements |

### Hue-Shift Shading

```javascript
shade(color, amount, warmth)
// Highlights shift toward yellow/white, shadows toward blue/purple
```

### Adaptive Outline

Outline color = saturated dark version of adjacent pixel (not uniform):
- Wood edge → deep brown, Metal edge → dark blue-gray, Plant edge → deep green

---

## Tileset Mode

When prompt contains **"tileset"**, **"sprite sheet"**, or **"asset pack"**, generate a complete sheet.

### How it works

1. **Shared engine** — All texture generators + post-processing in one file
2. **Shared palette** — One `P` object with all color sets, used by every asset
3. **Asset factories** — Each asset type is a function returning a standalone `PB`:
   ```javascript
   function mkBush(w, h, rx, ry) {
     const p = new PB(w, h);
     texFoliage(p, w/2, h/2, rx, ry, P.fol, P.folDk, .82);
     return pp(p, 0.5, 0.3);
   }
   ```
4. **Composition** — `blit()` all asset PBs onto a master canvas at specific positions
5. **Render** — Master canvas at 3x scale, neutral gray `#c0c0b8` background

### Theme → Asset List

| Theme | Hero | Structures | Nature | Terrain | Details |
|-------|------|------------|--------|---------|---------|
| Medieval village | Cottage | Fences, crates, well | Bushes, trees, flowers | Cobblestone, rocks | Grass, stumps, pebbles |
| Castle | Keep | Marble floors, arches, banners | Courtyard garden | Stone tiles, walls | Torches, shields |
| Market / Bazaar | Market tent | Stalls, weapon rack, potion cart | Potted plants | Cobblestone, carpet | Fruits, coins, banners |
| Fishing village | Warehouse | Barrels, nets, drying rack | Seaweed, coral | Dock planks, water | Fish, crates, rope |
| Dungeon | Castle gate | Torches, doors, chests | Moss, mushrooms | Stone floors, pits | Bones, chains, potions |
| Dark fantasy | Corrupted tower | Bone altars, cursed trees | Crystals, dark roots | Cracked stone, void | Webs, candles, runes |
| Forest | Oak tree | Campfire, tent, cabin | Bushes, ferns | Dirt paths, streams | Mushrooms, berries |
| Farm | Barn | Fences, troughs, windmill | Crops, haystacks | Dirt/grass, pond | Chickens, tools |
| Desert | Pyramid | Sand-brick walls, clay pots | Cacti, palms | Sand, cracked earth | Bones, gems, urns |
| Snow mountain | Lodge | Sleds, ice blocks | Snow pines, frozen bushes | Snow, ice tiles | Icicles, snowman |
| Cyberpunk | Neon building | Vending machines, holograms | None (urban) | Asphalt, grating | Neon signs, cables |
| Japanese garden | Pagoda | Torii gate, stone lantern | Cherry trees, bamboo | Stone path, koi pond | Koi, lotus, lantern |
| Pirate | Ship / dock | Cannons, treasure chest | Palm trees | Dock planks, water | Flags, rum bottles |
| Space station | Airlock | Terminals, cryo-pods | Alien flora | Metal floors, pipes | Monitors, wires |

**Assembly recipes:**
```
cottage = texStone(walls) + texRoof(roof) + texWood(door_frame) + pp()
market_stall = wood_frame + texCloth(awning) + blitAlpha(fruit_crate×4)
weapon_rack = wood_frame + blitAlpha(sword, texMetal) + blitAlpha(shield)
```

Aim for **25–40 assets** per tileset.

### Tileset Examples

| Prompt | Assets generated |
|--------|-----------------|
| `tileset, medieval village` | Cottage, fences×3, crates×2, bushes×6, cobblestone tiles×6, birdhouses×2, flowers×6, stumps×3, rocks×5, grass×6 |
| `tileset, dungeon crawler` | Gate, torch×2, chest, barrel, door×2, moss tiles, floor tiles×4, wall pieces×4, potions×3, bones, chains |

---

## Examples

### Input: `8x8, treasure chest`

```javascript
palette: { 'B':'#8b4513', 'L':'#cd853f', 'G':'#d4af37', 'D':'#5c2d00' },
pixels: [
  '........',
  '.BBBBBB.',
  'BLLLLLLD',
  'BLGGGLLB',
  'BBBBBBBB',
  'BLLLLLLD',
  'BLLLLLLD',
  'BDDDDDDB',
]
```

Rendered with CSS box-shadow. No animation (static item).

### Input: `16x24, ancient Chinese swordsman`

Full character with flowing hair, robe, sword. Grid renderer. Auto-adds: breathe, gleam (sword), wind (hair), fade-in. Dark background, no atmosphere.

### Input: `192x128, Egyptian pyramids at night`

Canvas-based procedural scene. FBM desert terrain, three pyramids (geometric triangles with block texture), starry sky, moon, atmospheric haze. Bayer dithering for sky gradient.

---

## Integration

### Claude Code

Reference this file or the project. Then just say:

```
pixel art, 32x32, a wizard with a glowing staff
```

### MCP / OpenClaw

```json
{
  "size": "32x32",
  "prompt": "a wizard with a glowing staff"
}
```

That's the entire input schema. Two fields.
