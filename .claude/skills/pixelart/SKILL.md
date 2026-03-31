---
name: pixelart
description: Generate pixel art as self-contained HTML files. Pure code, zero dependencies. Supports 8x8 icons to full landscape scenes.
argument-hint: [size] [prompt]
---

Generate a pixel art HTML file based on the user's request: $ARGUMENTS

Parse the input to extract **size** (e.g. `16x16`, `32x32`, `192x128`) and **prompt** (the subject to draw). If size is omitted, default to `16x16`.

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

### Prompt → Rendering Method

| Prompt implies...              | Method                                 |
|-------------------------------|----------------------------------------|
| Simple icon / item / emoji     | **CSS box-shadow** (zero JS, static)   |
| Character / sprite             | **CSS Grid + JS** (per-pixel control)  |
| "animated", movement, effects  | **Grid + animation JS**                |
| Landscape, scene, environment  | **Canvas** (procedural generation)     |

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

### Atmosphere Implementation

| Element | CSS/JS                                                     |
|---------|-------------------------------------------------------------|
| stars   | Random 1-2px dots, opacity twinkle                          |
| moon    | `border-radius:50%` + radial-gradient + box-shadow glow    |
| rain    | 1px divs, fast vertical translateY                          |
| petals  | `border-radius:50% 0 50% 0`, rotate + fall                 |
| mist    | Translucent gradient div, slow horizontal drift             |
| embers  | Small dots rising with box-shadow glow                      |
