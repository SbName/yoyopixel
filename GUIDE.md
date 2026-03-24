# 纯代码像素画 — 实现指南

用 HTML/CSS/JavaScript 从零绘制像素画，无需任何图片资源。

---

## 1. 数据表示：用字符串描述像素

核心思路是把图像"文本化"。每个字符代表一个像素的颜色：

```javascript
// 调色板：字符 → 颜色
palette: {
  '.': 'transparent',  // 透明（保留字符，必须用 '.'）
  'H': '#0f0f1a',      // 头发
  'S': '#f5d0a0',      // 皮肤
  'E': '#0a0a0a',      // 眼睛
  'B': '#1a3050',      // 蓝袍
}

// 点阵数据：每行字符串 = 一行像素
pixels: [
  '.......GG.......',  // 金簪
  '....hhSSSSSh....',  // 额头
  '....hSESSEShh...',  // 眼睛
]
```

好处：直观可读，像在画 ASCII art。修改一个字符就能改一个像素。

---

## 2. 尺寸预设体系

| 预设名     | 尺寸    | 推荐调色数 | 适用场景                        |
|-----------|---------|-----------|-------------------------------|
| `micro`   | 8×8     | 3–5       | 图标、emoji、极简物品             |
| `mini`    | 12×12   | 5–8       | 简单角色、道具、符号              |
| `small`   | 16×16   | 8–12      | 标准精灵、头像                   |
| `medium`  | 24×24   | 10–16     | 细节角色、肖像                   |
| `large`   | 32×32   | 12–20     | 复杂角色、完整场景                |
| `xlarge`  | 48×48   | 15–25     | 高精度插画                      |
| `portrait`| 16×24   | 10–16     | 竖版角色全身像                   |
| `wide`    | 32×16   | 8–15      | 横幅、风景、UI元素               |
| `scene`   | 48×32   | 15–25     | 角色 + 环境                     |

### 构图比例参考

| 画布尺寸 | 头部大小 | 身体比例 | 边框留白 |
|---------|---------|---------|---------|
| 8×8     | 4×4     | 1:1     | 0–1px   |
| 12×12   | 5×5     | 1:1.2   | 1px     |
| 16×16   | 6×6     | 1:1.5   | 1–2px   |
| 24×24   | 8×8     | 1:2     | 2px     |
| 32×32   | 10×10   | 1:2.2   | 2–3px   |
| 48×48   | 14×14   | 1:2.5   | 3–4px   |

---

## 3. 精细度等级

| 等级       | 颜色数 | 阴影处理 | 描述                                    |
|-----------|-------|---------|----------------------------------------|
| `minimal` | 3–5   | 无      | 纯平涂，剪影优先，图标化                   |
| `standard`| 6–10  | 基础    | 每种材质 1 个阴影色，特征清晰               |
| `detailed`| 10–16 | 完整    | 高光 + 阴影，次级细节，质感暗示             |
| `ultra`   | 16–25 | 高级    | 多层阴影，抖动渐变，边缘抗锯齿              |

### 着色策略

```
minimal:   纯平涂，不加阴影
standard:  每种材质 = 基色 + 1个暗色，光源左上
detailed:  每种材质 = 高光 + 基色 + 暗色，光源一致
ultra:     2+ 层暗色，边缘光，渐变区域用抖动（dithering）
```

---

## 4. 渲染方式一：CSS Grid

把每个字符渲染成一个 `<div>`，排进 CSS Grid：

```javascript
const SCALE = 8; // 每像素 8 屏幕像素

grid.style.gridTemplateColumns = `repeat(${width}, ${SCALE}px)`;

for (let y = 0; y < height; y++) {
  for (let x = 0; x < width; x++) {
    const ch = pixels[y][x];
    const color = palette[ch] || 'transparent';
    const div = document.createElement('div');
    div.style.width = SCALE + 'px';
    div.style.height = SCALE + 'px';
    div.style.background = color;
    grid.appendChild(div);
  }
}
```

原理：N 列的 Grid 容器 → 每个字符变成一个色块 → 自动排列成像素画。

## 5. 渲染方式二：纯 CSS box-shadow

一个 1×1px 的元素，通过 `box-shadow` 在不同偏移位置"投影"出每个像素：

```css
.pixel-art {
  width: 1px;
  height: 1px;
  transform: scale(6);
  transform-origin: top left;
  image-rendering: pixelated;
  background: transparent;
  box-shadow:
    7px 0px #d4af37,    /* (7,0) 金色 */
    5px 5px #f5d0a0,    /* (5,5) 肤色 */
    6px 7px #0a0a0a;    /* (6,7) 黑色 */
}
```

原理：`box-shadow` 本质是在指定偏移处复制一份元素。1px 元素的 shadow = 1 个像素点。`scale()` 统一放大。

## 6. 两种方式对比

| | CSS Grid | CSS box-shadow |
|---|---|---|
| 是否需要 JS | 需要（生成 DOM） | 不需要 |
| 可控性 | 高，每个像素是独立 DOM 元素 | 低，所有像素是一个属性 |
| 动画能力 | 可以单独动画每个像素 | 只能整体动画 |
| 适合场景 | 需要交互/动画的像素画 | 纯展示、轻量场景 |

---

## 7. 像素缩放体系

| 倍数  | 像素大小 | 适用场景            |
|------|---------|-------------------|
| 1x   | 1px     | 缩略图、拼图集       |
| 2x   | 2px     | 紧凑内联显示        |
| 4x   | 4px     | 小型展示、画廊网格    |
| 6x   | 6px     | 标准展示（默认）     |
| 8x   | 8px     | 大型展示、主角       |
| 10x  | 10px    | 超大展示、海报       |
| auto | 计算得出 | 自适应容器尺寸       |

auto 模式计算方式：
```javascript
const scale = Math.floor(Math.min(
  containerWidth / artWidth,
  containerHeight / artHeight
));
```

---

## 8. 动画效果

有了像素级控制（CSS Grid 方式），动画就是修改单个像素的属性：

### 可用动画

| 动画        | 描述                   | 实现方式                                |
|------------|------------------------|---------------------------------------|
| `breathe`  | 呼吸微动（上下）          | `translateY(0 → -2px)`, 4s 循环        |
| `gleam`    | 金属/刀剑闪光            | 逐像素 `brightness(1→3→1)`, 自上而下     |
| `wind`     | 头发/衣物风动            | `sin(time + x*0.5)` 控制亮度            |
| `blink`    | 眨眼                   | 眼睛像素 opacity 周期切换                |
| `float`    | 悬浮漂移                | `translateY` 慢速正弦运动               |
| `fade-in`  | 逐像素入场              | 对角线延迟 `opacity 0→1`               |
| `glow`     | 发光脉动                | `box-shadow` 大小周期变化               |
| `sparkle`  | 随机闪烁                | 随机选取像素短暂高亮                     |

### 示例 — 剑光闪烁

```javascript
const swordPixels = allPixels.filter(p => p.ch === 'A');
swordPixels.sort((a, b) => a.y - b.y);

swordPixels.forEach((p, i) => {
  setTimeout(() => {
    p.el.style.background = '#ffffff';
    p.el.style.boxShadow = '0 0 6px 2px rgba(255,255,255,0.6)';
    setTimeout(() => {
      p.el.style.background = p.baseColor;
      p.el.style.boxShadow = 'none';
    }, 200);
  }, i * 80);
});
```

---

## 9. 场景氛围系统

| 元素         | CSS 实现                                       |
|-------------|-----------------------------------------------|
| `moon`      | `border-radius: 50%` + `radial-gradient` + `box-shadow` 光晕 |
| `stars`     | 随机定位 1-2px 圆点 + `opacity` 闪烁动画              |
| `petals`    | `border-radius: 50% 0 50% 0` 花瓣形 + 下落旋转动画    |
| `snow`      | 小圆点 + 随机漂落路径 + 水平摆动                      |
| `rain`      | 细长矩形 + 高速垂直下落                              |
| `fireflies` | 小圆点 + `box-shadow` 发光 + 随机游走路径            |
| `mist`      | 半透明渐变 div + 缓慢水平位移                        |
| `moonray`   | 倾斜半透明条形 div，模拟光柱                          |
| `clouds`    | 多层半透明圆角矩形 + 缓慢平移                        |

---

## 10. 调色板设计规范

### 字符映射约定

| 字符 | 含义            | 示例颜色     |
|-----|----------------|------------|
| `.` | 透明（保留）      | transparent |
| `S` | 皮肤 Skin       | #f5d0a0    |
| `s` | 皮肤阴影         | #ddb080    |
| `H` | 头发 Hair (暗)   | #0f0f1a    |
| `h` | 头发 (中)        | #1a1a2e    |
| `i` | 头发高光         | #2a2a45    |
| `E` | 眼睛 Eyes       | #0a0a0a    |
| `M` | 嘴 Mouth        | #c96b5e    |
| `B` | 衣体 Body (暗)   | #1a3050    |
| `b` | 衣体 (亮)        | #2a5a8a    |
| `W` | 白色/亮色        | #e0e0e0    |
| `R` | 红色 Red        | #e94560    |
| `G` | 金色 Gold       | #d4af37    |
| `A` | 金属/刃 Arm     | #c8ccd4    |
| `D` | 暗色 Dark       | #151520    |

大写 = 暗色/基色，小写 = 亮色/高光 是推荐约定（非强制）。

### 色彩风格参考

```
冷色调角色:  #0f0f1a  #1a1a2e  #1a3050  #2a5a8a  #c8ccd4
暖色调角色:  #3d2b1f  #8b4513  #d4a574  #f0c08a  #fff3e0
赛博朋克:   #0a0a2e  #ff0066  #00ffcc  #ffcc00  #ff6600
自然森林:   #1a3c1a  #2d5a2d  #4a8c4a  #7ab87a  #c8e8c8
幻想魔法:   #1a0a3c  #4a1a8c  #8c3cdc  #c87cff  #e8b8ff
```

---

## 11. 设计流程

```
确定尺寸预设 (如 small = 16×16)
    ↓
选择精细度等级 (如 detailed = 10-16色)
    ↓
设计调色板 (用字符映射约定)
    ↓
逐行用字符"画"出像素 ← 这步最花时间
    ↓
选择渲染方式 (Grid 或 box-shadow)
    ↓
设置像素缩放 (默认 6x)
    ↓
加动画和场景氛围
```

### 设计技巧

- **先画轮廓剪影，再填充细节** — 确保整体形状可辨认
- **头部比例适当放大** — 像素画常见做法，强化辨识度
- **用 2-3 个深浅色表现立体感** — 如深蓝 `B` + 浅蓝 `b` = 袍子褶皱
- **善用透明像素** — `.` 不画东西，但负空间同样决定形状
- **对称但不完全对称** — 适当打破对称（飘发、武器），让角色更生动

---

## 12. 项目文件

```
pixelai/
├── pixelart-skill.md   # Skill 定义 — 参数规范 & 集成接口
├── GUIDE.md            # 本文档 — 实现原理 & 设计指南
├── index.html          # 画廊演示（多尺寸 + 两种渲染方式）
└── swordsman.html      # 场景演示（角色动画 + 月夜氛围）
```
