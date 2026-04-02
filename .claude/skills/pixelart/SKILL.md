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

### Available Procedural Textures

| Texture | Algorithm | Use for |
|---------|-----------|---------|
| `texStone` | Voronoi cells + mortar joints | Stone walls, cobblestone, rock |
| `texTiles` | Offset grid + overlap shading | Roof tiles, floor tiles |
| `texStucco` | Noisy flat fill | Plaster, stucco, painted walls |
| `texWood` | Sin-wave grain pattern | Wood beams, planks, furniture |
| `texPlanks` | Horizontal boards + edge lines | Wood floors, chest body, doors |
| `texBark` | Multi-frequency ridges + cracks | Tree trunks, branches |
| `texFoliage` | Fractal blob + irregular edges | Bushes, tree canopy, hedges |
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

### Post-Processing Pipeline (Multi-Pass v2)

```
pp(pb, { ao: 0.12, light: 0.1, specular: [{cx,cy,r,intensity}], outline: 0.28 })
```

1. **`passAO(pb, strength)`** — Ambient occlusion: darken pixels at region boundaries (where neighboring colors differ significantly)
2. **`passLight(pb, strength)`** — Directional light from top-left
3. **`passSpecular(pb, specMap)`** — Add specular highlights at specified points (for metal, crystal, glass)
4. **`passOutline(pb, amount)`** — Darken pixels adjacent to empty space

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

**Assembly pattern** — Build complex assets from layered sub-parts:
```
Assembly = base_structure + blitAlpha(items, contactShadow) + decoration
Example: fruit_stall = wood_frame + cloth_awning(texCloth) + blitAlpha(fruit_crate) × 4
```

### Shape Primitives (Expanded)

| Primitive | Usage |
|-----------|-------|
| `inE(x,y, cx,cy, rx,ry)` | Ellipse — body parts, rocks, barrels |
| `inTri(x,y, x1,y1, x2,y2, x3,y3)` | Triangle — roofs, crystal shards |
| `inPoly(x,y, vertices[])` | Arbitrary polygon — complex silhouettes |
| `inRR(x,y, x1,y1, x2,y2, r)` | Rounded rectangle — crates, signs, shields |
| `distY(y, x, freq, amp)` | Sinusoidal Y warp — cloth edges, organic forms |

### Per-Material Shading Guidance

| Material | Tones | Shading pattern |
|----------|-------|----------------|
| Stone | 5 | deep mortar → stone shadow → base → light → highlight chip |
| Wood | 4 | grain dark → grain base → grain light → edge highlight |
| Cloth | 5 | fold valley → fold shadow → base → fold light → fold peak |
| Metal | 6 | dark ambient → base → reflected mid → reflected light → specular edge → specular peak |
| Crystal | 5 | core glow → base facet → facet edge → bright facet → specular point |
| Bone | 4 | crack shadow → aged base → surface light → dry highlight |

### Asset Size Minimums

| Asset type | Minimum size | Recommended |
|-----------|-------------|-------------|
| Hero building | 64px | 80–120px |
| Furniture / structures | 32px | 40–56px |
| Containers (crates, barrels) | 20px | 24–32px |
| Small items (potions, food) | 12px | 16–20px |
| Detail items (pebbles, coins) | 8px | 8–12px |

### Rendering

Uses `<canvas>` with `PixelBuffer` class (RGBA). Each pixel is individually addressable with alpha channel. Rendered at configurable scale (3–6x typical) with `image-rendering: pixelated`.

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
  return pp(p);  // auto-shade + outline
}

// Variant factory — type parameter controls shape
function mkFence(type) {  // 'panel' | 'gate' | 'post'
  const p = new PB(22, 34);
  // posts, beams, iron bands based on type
  return pp(p);
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
| **Market / Bazaar** | Market tent (texCloth awning) | Stalls with goods, weapon rack, potion cart | Potted plants, flowers | Cobblestone, carpet (texWoven) | Fruits, bottles, coins, banners |
| **Fishing village** | Wooden warehouse | Shelf racks, barrels, nets, drying rack | Seaweed, coral | Dock planks, water tiles | Fish, crates, sacks, rope, anchor |
| **Dungeon** | Castle gate | Torches, doors, chests, barrels | Moss, vines, mushrooms | Stone floor tiles, walls, pits | Bones (texBone), chains, potions |
| **Dark fantasy** | Corrupted tower | Spider structures, bone altars, cursed trees | Crystal growths (texCrystal), dark roots (texCoral) | Cracked stone, void tiles | Webs, candles, skull, runes, glow particles |
| **Tropical / Tribal** | Totem pillar, tiki hut | Yurt/tent (texWoven), palm shelter | Palm trees, coral (texCoral), shells | Sand, beach tiles, shallow water | Tiki masks, seashells, rope (texRope), pearls |
| **Forest** | Large oak tree | Campfire, tent (texCloth), log cabin | Bushes, ferns, saplings | Dirt paths, streams, boulders | Mushrooms, berries, acorns |
| **Farm** | Barn | Fences, troughs, windmill | Crops, haystacks, flowers | Dirt/grass tiles, pond | Chickens, tools, baskets |

**Assembly recipes** for composite assets:
```
market_stall = wood_frame + texCloth(awning, stripes) + blitAlpha(fruit_crate×4) + contact_shadows
weapon_rack = wood_frame + blitAlpha(sword, texMetal) + blitAlpha(axe) + blitAlpha(shield)
shelf_unit = wood_shelves + blitAlpha(sack, texCloth) + blitAlpha(barrel) + blitAlpha(crate)
fishing_net = rope_frame(texRope) + blitAlpha(hanging_fish×4)
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
