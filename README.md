<p align="center">
  <img src="https://img.shields.io/badge/YoYoPixel-Pure_Code_Pixel_Art-e94560?style=for-the-badge&labelColor=1a1a2e" alt="YoYoPixel">
</p>

<h3 align="center">Pixel art without GPU. Any LLM can draw.</h3>

<p align="center">
  No GPU. No image model. No multimodal. No dependencies.<br>
  <b>Any text-only LLM + size + prompt → pixel art HTML</b>
</p>

<p align="center">
  <a href="#install-as-claude-code-skill"><img src="https://img.shields.io/badge/Claude_Code-Skill-00c8ff?style=flat-square" alt="Claude Code"></a>
  <a href="pixelart-skill.md"><img src="https://img.shields.io/badge/MCP-Compatible-00ff88?style=flat-square" alt="MCP"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-d4af37?style=flat-square" alt="MIT"></a>
</p>

---

## What is YoYoPixel?

**Text-only LLMs can generate images — no GPU, no DALL-E, no multimodal model needed.**

YoYoPixel is a skill/prompt framework that teaches any LLM (Claude, GPT, Gemini, Llama, Qwen...) to generate pixel art as pure HTML/CSS/JS code. The LLM writes structured text data, the browser renders it as pixel art.

```
Traditional: prompt → GPU → image model → PNG
YoYoPixel:   prompt → any text LLM → HTML code → pixel art in browser
```

```
/pixelart 32x32, a wizard with a glowing staff
```

The AI auto-decides everything: palette, detail level, rendering method, shading, animations, atmosphere. No configuration needed.

### Capabilities at a glance

| Size Range | What it can do |
|-----------|----------------|
| **8×8** | Icons, emojis, game items (sword, potion, key, chest) |
| **12–16px** | Character sprites, avatars, simple creatures |
| **24–32px** | Detailed characters with outfits, weapons, accessories |
| **48px+** | Complex illustrations with sub-pixel shading |
| **96–256px** | Full procedural landscapes, cityscapes, world landmarks |

---

## Install as Claude Code Skill

### Option 1: Project-level (recommended)

```bash
# Clone
git clone https://github.com/SbName/yoyopixel.git

# Copy skill into your project
cp -r yoyopixel/.claude/skills/pixelart YOUR_PROJECT/.claude/skills/
```

### Option 2: Global (all projects)

```bash
cp -r yoyopixel/.claude/skills/pixelart ~/.claude/skills/
```

### Use it

```
/pixelart 16x16, a fire mage with staff
/pixelart 32x32, cyberpunk cat with neon goggles
/pixelart 192x128, mountain sunset with lake reflection
/pixelart 24x24, ancient Chinese swordsman in moonlight
```

Two inputs: **size** + **prompt**. That's the entire interface.

---

## Showcases

> Open any `.html` file in a browser. No build step needed.

### Characters & Scenes

<table>
<tr>
<td align="center"><b>Swordsman</b><br><sub>16×24 · Moonlit scene</sub><br><img src="assets/swordsman.png" width="380"></td>
<td align="center"><b>Cyberpunk Girl</b><br><sub>32×32 · Neon rain</sub><br><img src="assets/cyberpunk.png" width="380"></td>
</tr>
<tr>
<td align="center"><b>Fire Dragon</b><br><sub>24×24 · Ember particles</sub><br><img src="assets/dragon.png" width="380"></td>
<td align="center"><b>Pixel Icons</b><br><sub>8×8 · 12 game icons</sub><br><img src="assets/icons.png" width="380"></td>
</tr>
</table>

### Procedural Generators

<table>
<tr>
<td align="center"><b>Landscape Generator</b><br><sub>FBM noise · 5 biomes · up to 256×144</sub><br><img src="assets/landscape.png" width="380"></td>
<td align="center"><b>World Wonders</b><br><sub>8 landmarks · Pyramids, Great Wall, Taj Mahal...</sub><br><img src="assets/wonders.png" width="380"></td>
</tr>
</table>

### Multi-Size Gallery

<p align="center">
<img src="assets/gallery.png" width="700">
</p>
<p align="center"><sub>8×8 icons → 12×12 sprites → 16×16 avatars → 24×24 characters → 32×32 detailed</sub></p>

---

## How It Works

### Small art (≤48px): String-array pixel data

Each character in a string = one pixel. Human-readable, AI-writable.

```javascript
const wizard = {
  width: 24, height: 24,
  palette: {
    '.': 'transparent',
    'H': '#2a1a4e',     // hat
    'S': '#f5d0a0',     // skin
    'R': '#6a3cbf',     // robe
  },
  pixels: [
    '........HHHH............',
    '......HHHHHHHH..........',
    '....HHHHHHHHHHH.........',
    '...HHHSSSSSSSHHH........',
  ]
};
```

Rendered via **CSS Grid** (per-pixel animation) or **CSS box-shadow** (zero JS, pure CSS).

### Large art (≥64px): Canvas-based procedural generation

```
FBM Noise → Terrain shape
Ridge Noise → Mountain ridgelines
Bayer 4×4 Dithering → Palette-limited gradients
Layered Rendering → Sky → Stars → Clouds → Mountains → Trees → Water → Mist
```

All algorithms run in-browser. No server, no API calls.

---

## For AI Platforms

### Claude Code

Install the skill, then:
```
/pixelart 32x32, a knight in silver armor
```

### MCP / OpenClaw

```json
{
  "size": "32x32",
  "prompt": "a knight in silver armor"
}
```

### What the AI auto-decides

| Input | AI figures out |
|-------|---------------|
| Size `8x8` | minimal detail, 3-5 colors, CSS box-shadow, no animation |
| Size `32x32` | detailed, 12-20 colors, CSS Grid, breathe + gleam + fade-in |
| Prompt mentions "night" | adds stars, moon, mist atmosphere |
| Prompt mentions "fire" | adds ember particles, glow effects |
| Prompt mentions "sword" | adds gleam animation on weapon pixels |
| Size `192x128` | canvas mode, FBM noise, procedural generation |

Full decision rules → [`pixelart-skill.md`](pixelart-skill.md)

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Rendering (small) | CSS Grid / CSS box-shadow |
| Rendering (large) | HTML5 Canvas + PixelBuffer |
| Terrain generation | FBM (Fractal Brownian Motion) noise |
| Mountain ridges | Ridge noise (abs-inverted FBM) |
| Gradient dithering | Bayer 4×4 ordered dithering |
| Animation | CSS @keyframes + requestAnimationFrame |
| Atmosphere | Pure CSS particles (rain, petals, embers, snow) |
| Output | Self-contained HTML (zero external dependencies) |

---

## Project Structure

```
pixelai/
├── .claude/skills/pixelart/
│   └── SKILL.md              ← Claude Code skill definition
│
├── README.md                 ← You are here
├── pixelart-skill.md         ← Full skill reference (AI decision rules)
├── GUIDE.md                  ← Technical implementation guide
├── LICENSE                   ← MIT
│
├── showcase.html             ★ All-in-one demo page
├── landscape.html            ★ Procedural landscape generator
├── wonders.html              ★ World wonders (8 landmarks)
├── index.html                ← Multi-size gallery
├── swordsman.html            ← Moonlit swordsman scene
├── cyberpunk.html            ← Neon cyberpunk scene
├── dragon.html               ← Fire dragon scene
└── icons.html                ← 8×8 icon collection
```

---

## License

[MIT](LICENSE) — use it however you want.
