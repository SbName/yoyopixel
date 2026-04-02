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

### Procedural Textures (12 types)

| Texture | Algorithm | Use for |
|---------|-----------|---------|
| `texStone` | Voronoi cells + mortar joints | Stone walls, cobblestone, rock surfaces |
| `texTiles` | Offset grid + overlap shading | Roof tiles, floor tiles, paving |
| `texStucco` | Noisy flat fill | Plaster, stucco, painted walls |
| `texWood` | Sin-wave grain pattern | Wood beams, door, furniture |
| `texPlanks` | Horizontal boards + edge lines | Plank floors, chest body, siding |
| `texBark` | Multi-frequency ridges + cracks | Tree trunks, branches |
| `texFoliage` | Fractal blob + irregular edges | Bushes, tree canopy, hedges |
| `texFur` | Directional strand pattern | Animal fur, hair |
| `texScales` | Overlapping crescent grid | Fish, reptile, dragon scales |
| `texFeathers` | V-shaft + barb layering | Bird plumage, wings |
| `texSpiral` | Logarithmic spiral + dome shading | Snail shell, decorative spiral |
| `texSmooth` | Gradient fill with noise | Smooth skin, fins, body |

### Post-Processing Pipeline

Applied after all regions are filled:

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

### Single Asset Examples

| Prompt | Engine produces |
|--------|----------------|
| `80x85, stone cottage with red roof` | Voronoi stone wall + offset-grid tiles + stucco gable + arched wood door + glass windows |
| `48x90, wizard tower at night` | Dark stone + purple conical roof + glowing windows with addGlow + flag |
| `50x40, fox sitting in grass` | Fur texture with per-zone direction angles + ellipse composite body |
| `44x26, koi fish in pond` | Scale overlay + gradient fins + water ripple background |

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
     return pp(p);  // auto-shade + outline per asset
   }
   ```
4. **Composition** — `blit()` all asset PBs onto a master canvas at specific positions
5. **Render** — Master canvas at 3x scale, neutral gray `#c0c0b8` background

### Theme → Asset List

| Theme | Hero | Structures | Nature | Terrain | Details |
|-------|------|------------|--------|---------|---------|
| Medieval village | Cottage | Fences, crates, well, birdhouse | Bushes, trees, flowers | Cobblestone, rocks, cliffs | Grass, stumps, pebbles |
| Dungeon | Castle gate | Torches, doors, chests, barrels | Moss, vines, mushrooms | Stone floors, walls, pits | Bones, chains, potions |
| Forest | Oak tree | Campfire, tent, log cabin | Bushes, ferns, saplings | Dirt paths, streams, boulders | Mushrooms, berries |
| Farm | Barn | Fences, troughs, windmill | Crops, haystacks | Dirt/grass tiles, pond | Chickens, tools, baskets |

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
