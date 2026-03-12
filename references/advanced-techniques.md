# Advanced SVG Techniques Reference

Complete cookbook for reference-grade SVGs. All filters use `color-interpolation-filters="linearRGB"` for physically accurate blending.

## Table of Contents
1. Filter Chain Cookbook
2. feTurbulence Parameter Guide
3. feComponentTransfer Color Grading
4. Material Simulation
5. Illustration Composition Templates
6. Atmospheric and Environmental Effects
7. Icons with Depth
8. Logos with Dimension
9. Animation (CSS + SMIL)
10. Data Visualizations
11. Patterns and Backgrounds
12. Power Features Reference
13. Character Construction Templates

---

## 1. Filter Chain Cookbook

All filters: `x="-20%" y="-20%" width="140%" height="140%" color-interpolation-filters="linearRGB"`.

### Drop Shadow (Blue-Tinted)
```xml
<filter id="drop-shadow" x="-20%" y="-20%" width="140%" height="140%"
  color-interpolation-filters="linearRGB">
  <feGaussianBlur in="SourceAlpha" stdDeviation="4" result="blur"/>
  <feOffset in="blur" dx="3" dy="5" result="offset"/>
  <feFlood flood-color="#1e1b4b" flood-opacity="0.30" result="color"/>
  <feComposite in="color" in2="offset" operator="in" result="shadow"/>
  <feMerge>
    <feMergeNode in="shadow"/>
    <feMergeNode in="SourceGraphic"/>
  </feMerge>
</filter>
```

### Contact Shadow (Tight, Under Objects)
Same as drop shadow but: `stdDeviation="1.5"`, `dx="0" dy="1"`, `flood-opacity="0.40"`.

### Cast Shadow (Projected, Softening)
Same pattern: `stdDeviation="6"`, `dx="4" dy="8"`, `flood-opacity="0.20"`.

### Soft Glow
```xml
<filter id="glow" x="-30%" y="-30%" width="160%" height="160%"
  color-interpolation-filters="linearRGB">
  <feGaussianBlur in="SourceGraphic" stdDeviation="8" result="blur"/>
  <feMerge>
    <feMergeNode in="blur"/>
    <feMergeNode in="SourceGraphic"/>
  </feMerge>
</filter>
```
For colored glow: add `<feColorMatrix type="matrix">` before merge to tint the blur.

### Inner Shadow
```xml
<filter id="inner-shadow" x="-10%" y="-10%" width="120%" height="120%"
  color-interpolation-filters="linearRGB">
  <feComponentTransfer in="SourceAlpha" result="inverse">
    <feFuncA type="table" tableValues="1 0"/>
  </feComponentTransfer>
  <feGaussianBlur in="inverse" stdDeviation="3" result="blur"/>
  <feOffset in="blur" dx="2" dy="3" result="offset"/>
  <feComposite in="offset" in2="SourceAlpha" operator="in" result="inner"/>
  <feFlood flood-color="#1e1b4b" flood-opacity="0.5" result="color"/>
  <feComposite in="color" in2="inner" operator="in" result="shadow"/>
  <feMerge>
    <feMergeNode in="SourceGraphic"/>
    <feMergeNode in="shadow"/>
  </feMerge>
</filter>
```

### Salt & Pepper Texture (Two-Layer Grain)
The professional method: "pepper" layer darkens via multiply, "salt" layer brightens via overlay. Both at low opacity for tactile quality:
```xml
<filter id="salt-pepper" x="0" y="0" width="100%" height="100%"
  color-interpolation-filters="linearRGB">
  <feTurbulence type="fractalNoise" baseFrequency="0.7" numOctaves="3"
    stitchTiles="stitch" result="noise"/>
  <feColorMatrix in="noise" type="saturate" values="0" result="mono"/>
  <!-- Pepper (shadow grain) -->
  <feBlend in="SourceGraphic" in2="mono" mode="multiply" result="pepper"/>
  <!-- Salt (highlight grain) -->
  <feBlend in="pepper" in2="mono" mode="overlay" result="salted"/>
  <!-- Control intensity -->
  <feComponentTransfer in="salted">
    <feFuncA type="linear" slope="1" intercept="0"/>
  </feComponentTransfer>
</filter>
```
Apply at element level for selective texturing. Use `mode="soft-light"` instead of `mode="overlay"` for subtler salt layer.

### Subtle Paper Texture
```xml
<filter id="paper" color-interpolation-filters="linearRGB">
  <feTurbulence type="fractalNoise" baseFrequency="0.9" numOctaves="4" result="noise"/>
  <feColorMatrix in="noise" type="saturate" values="0" result="gray"/>
  <feComponentTransfer in="gray" result="subtle">
    <feFuncR type="linear" slope="0.12" intercept="0.44"/>
    <feFuncG type="linear" slope="0.12" intercept="0.44"/>
    <feFuncB type="linear" slope="0.12" intercept="0.44"/>
  </feComponentTransfer>
  <feBlend in="SourceGraphic" in2="subtle" mode="multiply"/>
</filter>
```

### Specular Lighting (Shiny Surface)
```xml
<filter id="specular" x="-10%" y="-10%" width="120%" height="120%"
  color-interpolation-filters="linearRGB">
  <feSpecularLighting in="SourceAlpha" surfaceScale="5" specularConstant="0.75"
    specularExponent="25" lighting-color="#ffffff" result="specular">
    <fePointLight x="250" y="100" z="300"/>
  </feSpecularLighting>
  <feComposite in="specular" in2="SourceAlpha" operator="in" result="lit"/>
  <feMerge>
    <feMergeNode in="SourceGraphic"/>
    <feMergeNode in="lit"/>
  </feMerge>
</filter>
```
`specularExponent`: 20–40 = glossy, 5–10 = matte. Light position: x,y in SVG coords, z = height above surface.

### Diffuse Lighting (Matte Surface)
```xml
<filter id="diffuse" color-interpolation-filters="linearRGB">
  <feDiffuseLighting in="SourceAlpha" surfaceScale="4" diffuseConstant="1"
    lighting-color="#ffe4c4" result="diffuse">
    <feDistantLight azimuth="225" elevation="45"/>
  </feDiffuseLighting>
  <feComposite in="diffuse" in2="SourceGraphic" operator="in" result="lit"/>
  <feBlend in="SourceGraphic" in2="lit" mode="multiply"/>
</filter>
```

### Emboss / Bevel
```xml
<filter id="emboss" color-interpolation-filters="linearRGB">
  <feConvolveMatrix order="3" kernelMatrix="-2 -1 0  -1 1 1  0 1 2"/>
</filter>
```
Sharpen: `kernelMatrix="0 -1 0  -1 5 -1  0 -1 0"`.

### Glassmorphism (Frosted Glass)
```xml
<filter id="frosted-glass" x="-20%" y="-20%" width="140%" height="140%"
  color-interpolation-filters="linearRGB">
  <feTurbulence type="fractalNoise" baseFrequency="0.8" numOctaves="3" result="noise"/>
  <feColorMatrix in="noise" type="matrix" result="soft"
    values="1 0 0 0 0  0 1 0 0 0  0 0 1 0 0  0 0 0 0.5 0"/>
  <feGaussianBlur in="SourceGraphic" stdDeviation="8" result="blurred"/>
  <feDisplacementMap in="blurred" in2="soft" scale="12"
    xChannelSelector="R" yChannelSelector="G" result="distorted"/>
  <feSpecularLighting in="soft" surfaceScale="2" specularConstant="0.6"
    specularExponent="30" lighting-color="#ffffff" result="shine">
    <fePointLight x="200" y="50" z="200"/>
  </feSpecularLighting>
  <feComposite in="shine" in2="SourceAlpha" operator="in" result="glass-shine"/>
  <feMerge>
    <feMergeNode in="distorted"/>
    <feMergeNode in="glass-shine"/>
  </feMerge>
</filter>
```

### Vignette
```xml
<radialGradient id="vignette-grad" cx="50%" cy="50%" r="60%">
  <stop offset="0%" stop-color="black" stop-opacity="0"/>
  <stop offset="100%" stop-color="black" stop-opacity="0.45"/>
</radialGradient>
<rect width="800" height="600" fill="url(#vignette-grad)" style="mix-blend-mode:multiply"/>
```

---

## 2. feTurbulence Parameter Guide

`baseFrequency` controls texture scale:
- **0.01–0.05** — large cloud-like patterns, terrain
- **0.05–0.2** — medium organic textures (water, marble, smoke)
- **0.2–0.5** — coarse grain (stone, sand)
- **0.5–0.8** — fine grain (paper, film noise) ← most useful for texture overlays
- **0.8–2.0** — very fine static

Two-value baseFrequency creates directional stretch:
- `"0.02 0.2"` — horizontal grain (wood grain)
- `"0.01 0.5"` — strong horizontal stretch (glitch effect)

`numOctaves`: 1 = rough, **3 = sweet spot**, 5+ = diminishing returns.

`type="fractalNoise"` vs `type="turbulence"`:
- `fractalNoise` — smoother, better for grain/texture overlays
- `turbulence` — more chaotic, better for clouds/smoke/water

`seed` — different integer = different random pattern. Change for variety.

`stitchTiles="stitch"` — ensures seamless tiling at boundaries. Always use for pattern fills.

---

## 3. feComponentTransfer Color Grading

Per-channel color manipulation for professional color grading, duotone effects, and tonal control.

### Increase Contrast
```xml
<filter id="contrast" color-interpolation-filters="linearRGB">
  <feComponentTransfer>
    <feFuncR type="linear" slope="1.5" intercept="-0.15"/>
    <feFuncG type="linear" slope="1.5" intercept="-0.15"/>
    <feFuncB type="linear" slope="1.5" intercept="-0.15"/>
  </feComponentTransfer>
</filter>
```

### Warm Color Shift (Sunset Feel)
```xml
<feComponentTransfer>
  <feFuncR type="linear" slope="1.1" intercept="0.05"/>
  <feFuncG type="linear" slope="0.95" intercept="0"/>
  <feFuncB type="linear" slope="0.85" intercept="-0.05"/>
</feComponentTransfer>
```

### Cool Color Shift (Moonlit Feel)
```xml
<feComponentTransfer>
  <feFuncR type="linear" slope="0.85" intercept="-0.05"/>
  <feFuncG type="linear" slope="0.9" intercept="0"/>
  <feFuncB type="linear" slope="1.15" intercept="0.05"/>
</feComponentTransfer>
```

### Posterize (Reduce Color Steps)
```xml
<feComponentTransfer>
  <feFuncR type="discrete" tableValues="0 0.25 0.5 0.75 1"/>
  <feFuncG type="discrete" tableValues="0 0.25 0.5 0.75 1"/>
  <feFuncB type="discrete" tableValues="0 0.25 0.5 0.75 1"/>
</feComponentTransfer>
```

### Duotone (Two-Color Map)
```xml
<filter id="duotone" color-interpolation-filters="linearRGB">
  <feColorMatrix type="saturate" values="0" result="gray"/>
  <feComponentTransfer in="gray">
    <feFuncR type="table" tableValues="0.1 0.9"/>   <!-- dark-red to light-red -->
    <feFuncG type="table" tableValues="0.0 0.6"/>   <!-- dark-green to light-green -->
    <feFuncB type="table" tableValues="0.3 0.95"/>   <!-- dark-blue to light-blue -->
  </feComponentTransfer>
</filter>
```

### Gamma Curve (Lighten Midtones)
```xml
<feComponentTransfer>
  <feFuncR type="gamma" amplitude="1" exponent="0.7" offset="0"/>
  <feFuncG type="gamma" amplitude="1" exponent="0.7" offset="0"/>
  <feFuncB type="gamma" amplitude="1" exponent="0.7" offset="0"/>
</feComponentTransfer>
```

---

## 4. Material Simulation

### Metal (Steel/Chrome)
High-contrast multi-stop gradient with sharp transitions. Add `spreadMethod="reflect"` for brushed-metal repeat:
```xml
<linearGradient id="steel" x1="0" y1="0" x2="0" y2="1" color-interpolation="linearRGB">
  <stop offset="0%" stop-color="#e8e8e8"/>
  <stop offset="20%" stop-color="#6b6b6b"/>
  <stop offset="35%" stop-color="#d4d4d4"/>
  <stop offset="50%" stop-color="#888"/>
  <stop offset="65%" stop-color="#e0e0e0"/>
  <stop offset="80%" stop-color="#555"/>
  <stop offset="100%" stop-color="#b0b0b0"/>
</linearGradient>
```
Add thin near-white specular lines along edges. Apply specular filter. Include reflected environment colors at 15–30% opacity.

### Gold
Same structure, warm tones:
- Highlights: #fff7d1, #ffd700, #ffec99
- Base: #daa520, #b8860b
- Shadows: #8b6914, #6b4c0a, #4a3508

### Glass / Transparent
```xml
<!-- Base shape with low opacity -->
<rect rx="12" width="200" height="300" fill="#88ccff" opacity="0.12"/>
<!-- Highlight streak (angled linear gradient) -->
<rect rx="8" x="15" y="8" width="50" height="240"
  fill="url(#glass-highlight)" opacity="0.5"/>
<!-- Edge darkening (inner shadow filter or dark border) -->
<rect rx="12" width="200" height="300" fill="none" stroke="#1e3a5f" stroke-width="1"
  opacity="0.3"/>
<!-- Reflected light on bottom edge -->
<ellipse cx="100" cy="285" rx="70" ry="10" fill="white" opacity="0.1"/>
```
`#glass-highlight`: linear gradient from `white 0%` at `stop-opacity="0.6"` to `white 100%` at `stop-opacity="0"`, with `gradientTransform="rotate(-15)"`.

### Wood
Two-value baseFrequency creates directional grain:
```xml
<filter id="wood-grain" color-interpolation-filters="linearRGB">
  <feTurbulence type="fractalNoise" baseFrequency="0.02 0.2" numOctaves="5" result="grain"/>
  <feColorMatrix in="grain" type="matrix" result="brown"
    values="0.4 0.3 0.1 0 0.3  0.3 0.2 0.1 0 0.15  0.1 0.1 0.05 0 0.05  0 0 0 1 0"/>
</filter>
```

### Water
- Multiple semi-transparent blue layers at opacity 0.2–0.5
- Thin concentric elliptical ripple lines (white, opacity 0.3)
- Vertically flipped desaturated reflections at 30–50% opacity
- `feTurbulence` + `feDisplacementMap` for wavy distortion
- Gradient: dark blue deep → mid cyan → light blue surface with 5+ stops

### Stone / Rock
- Base gray-brown gradient
- Heavy noise: `baseFrequency="0.35"`, multiply blend, opacity 0.6
- Irregular polygon shapes
- Thin dark crack lines with slight organic curves

### Fabric / Cloth
- Alternating light-dark-light gradient bands simulate folds
- Very fine noise: `baseFrequency="0.8"`, soft-light blend
- Curved shadow shapes following drape direction

---

## 5. Illustration Composition Templates

### Landscape Scene (7 Layers)
```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 800 500"
  role="img" aria-labelledby="t d">
  <title id="t">Landscape Title</title>
  <desc id="d">Description</desc>
  <defs>
    <!-- sky gradient (6+ stops), sun glow radial, shadow filter, haze gradient,
         noise filter, vignette gradient -->
  </defs>

  <!-- L1: Sky with multi-stop gradient -->
  <rect width="800" height="500" fill="url(#sky)"/>

  <!-- L2: Sun/Moon with soft glow -->
  <circle cx="600" cy="120" r="40" fill="#fbbf24"/>
  <circle cx="600" cy="120" r="90" fill="url(#sun-glow)" opacity="0.4"/>

  <!-- L3: Far mountains (low sat, high lightness, blue-shifted) -->
  <path d="M0,350 Q200,220 400,310 Q550,260 800,320 L800,500 L0,500Z"
    fill="#94a3b8" opacity="0.4"/>

  <!-- L4: Mid mountains (medium sat) -->
  <path d="M0,380 Q150,290 300,350 Q450,310 600,340 L800,370 L800,500 L0,500Z"
    fill="#64748b" opacity="0.6"/>

  <!-- L5: Near hills (full sat, detail) -->
  <path d="M0,420 Q200,360 400,400 Q600,375 800,410 L800,500 L0,500Z"
    fill="url(#hill-gradient)"/>

  <!-- L6: Foreground details (trees, rocks with shadows) -->
  <g id="trees" filter="url(#drop-shadow)">
    <use href="#tree" x="120" y="370" width="60" height="90"/>
    <use href="#tree" x="350" y="380" width="45" height="70" opacity="0.85"/>
  </g>

  <!-- L7: Atmospheric haze between layers -->
  <rect y="280" width="800" height="220" fill="#bfdbfe" opacity="0.12"/>

  <!-- L8: Vignette overlay -->
  <rect width="800" height="500" fill="url(#vignette-grad)"/>
</svg>
```

### Object Study (Full Five-Zone Lighting)
```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 400 400">
  <defs>
    <radialGradient id="obj" fx="0.35" fy="0.3" cx="45%" cy="45%" r="55%"
      color-interpolation="linearRGB">
      <stop offset="0%" stop-color="#fff7ed" stop-opacity="0.8"/>  <!-- specular area -->
      <stop offset="15%" stop-color="#fb923c"/>                     <!-- light area -->
      <stop offset="50%" stop-color="#ea580c"/>                     <!-- half-tone -->
      <stop offset="85%" stop-color="#7c2d12"/>                     <!-- form shadow -->
      <stop offset="100%" stop-color="#431407"/>                    <!-- deep shadow -->
    </radialGradient>
    <filter id="soft"><feGaussianBlur stdDeviation="4"
      color-interpolation-filters="linearRGB"/></filter>
    <clipPath id="obj-clip"><circle cx="200" cy="200" r="85"/></clipPath>
  </defs>

  <!-- Background gradient -->
  <rect width="400" height="400" fill="url(#bg-grad)"/>

  <!-- Cast shadow on surface -->
  <ellipse cx="215" cy="340" rx="75" ry="12" fill="#1e1b4b" opacity="0.2"
    filter="url(#soft)"/>

  <!-- Contact shadow -->
  <ellipse cx="205" cy="325" rx="50" ry="5" fill="#1e1b4b" opacity="0.35"/>

  <!-- Main object with gradient -->
  <circle cx="200" cy="210" r="85" fill="url(#obj)"/>

  <!-- Form shadow overlay (cool-shifted) -->
  <ellipse cx="225" cy="235" rx="70" ry="75" fill="#3b1764" opacity="0.2"
    style="mix-blend-mode:multiply" clip-path="url(#obj-clip)"/>

  <!-- Highlight -->
  <ellipse cx="175" cy="180" rx="35" ry="25" fill="#fff7ed" opacity="0.35"
    filter="url(#soft)"/>

  <!-- Specular dot -->
  <circle cx="170" cy="172" r="7" fill="white" opacity="0.85"/>

  <!-- Reflected/bounced light on shadow edge -->
  <ellipse cx="230" cy="250" rx="25" ry="35" fill="#fef3c7" opacity="0.12"
    clip-path="url(#obj-clip)"/>

  <!-- Texture overlay -->
  <circle cx="200" cy="210" r="85" fill="none" filter="url(#salt-pepper)"
    opacity="0.08"/>
</svg>
```

### Stylized Character
- Head: circle/oval with radial gradient (skin tone, focus offset toward light)
- Body: rounded rect with fold-simulating linear gradient
- Eyes: small dark circles with white specular dots (2px)
- Hair: overlapping ellipses/paths with gradient fills
- Clothing: shapes with alternating light-dark gradient bands for folds
- Shadow under feet: contact shadow ellipse

For every body part: base gradient → form shadow overlay → highlight overlay → specular dot = dimensional form.

---

## 6. Atmospheric and Environmental Effects

### Light Rays (God Rays)
```xml
<g opacity="0.12" style="mix-blend-mode:screen">
  <polygon points="500,80 280,500 340,500" fill="#fbbf24"/>
  <polygon points="500,80 400,500 470,500" fill="#fbbf24"/>
  <polygon points="500,80 550,500 640,500" fill="#fbbf24"/>
  <polygon points="500,80 700,500 780,500" fill="#fbbf24"/>
</g>
```

### Fog / Mist
Semi-transparent white gradient, masked to fade at edges:
```xml
<defs>
  <mask id="fog-mask">
    <linearGradient id="fog-fade" x1="0" y1="0" x2="0" y2="1">
      <stop offset="0%" stop-color="black"/>
      <stop offset="30%" stop-color="white"/>
      <stop offset="70%" stop-color="white"/>
      <stop offset="100%" stop-color="black"/>
    </linearGradient>
    <rect width="800" height="200" fill="url(#fog-fade)"/>
  </mask>
</defs>
<rect y="300" width="800" height="200" fill="white" opacity="0.25"
  mask="url(#fog-mask)"/>
```

### Stars (with Twinkling)
```xml
<style>
  @keyframes twinkle {
    0%, 100% { opacity: 0.3; }
    50% { opacity: 1; }
  }
  .star { animation: twinkle 3s ease-in-out infinite; transform-box: fill-box; }
</style>
<g id="stars">
  <circle class="star" cx="50" cy="30" r="1" fill="white" style="animation-delay:0s"/>
  <circle class="star" cx="200" cy="55" r="1.5" fill="white" style="animation-delay:1.2s"/>
  <circle class="star" cx="400" cy="40" r="0.8" fill="white" style="animation-delay:0.5s"/>
  <!-- Vary: size 0.5-2, delay 0-4s -->
</g>
```

### Rain
```xml
<g opacity="0.25" stroke="white" stroke-width="1" stroke-linecap="round">
  <line x1="100" y1="0" x2="88" y2="60"/>
  <line x1="250" y1="20" x2="238" y2="80"/>
  <!-- Many more, slight angle, varied lengths 40-80 -->
</g>
```

### Clouds
```xml
<g filter="url(#soft)">
  <ellipse cx="200" cy="100" rx="80" ry="35" fill="white" opacity="0.9"/>
  <ellipse cx="155" cy="108" rx="55" ry="30" fill="white" opacity="0.85"/>
  <ellipse cx="245" cy="106" rx="60" ry="28" fill="white" opacity="0.85"/>
  <ellipse cx="200" cy="118" rx="90" ry="25" fill="white" opacity="0.8"/>
</g>
```

---

## 7. Icons with Depth

At 24×24: use 2–3 stop gradients instead of flat fills. `paint-order="stroke fill"` for halo effects.

At 64×64+ (app icons):
```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64">
  <defs>
    <linearGradient id="bg" x1="0" y1="0" x2="0" y2="1" color-interpolation="linearRGB">
      <stop offset="0%" stop-color="#818cf8"/>
      <stop offset="50%" stop-color="#6366f1"/>
      <stop offset="100%" stop-color="#4338ca"/>
    </linearGradient>
    <filter id="ico-shadow" x="-20%" y="-20%" width="140%" height="140%"
      color-interpolation-filters="linearRGB">
      <feGaussianBlur in="SourceAlpha" stdDeviation="2"/>
      <feOffset dy="2"/>
      <feFlood flood-color="#1e1b4b" flood-opacity="0.25"/>
      <feComposite operator="in" in2="SourceAlpha"/>
      <feMerge><feMergeNode/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
  </defs>
  <rect width="64" height="64" rx="14" fill="url(#bg)" filter="url(#ico-shadow)"/>
  <!-- Top shine -->
  <rect x="4" y="4" width="56" height="28" rx="10" fill="white" opacity="0.12"/>
  <!-- Icon glyph -->
  <g transform="translate(32,32)" fill="none" stroke="white" stroke-width="2.5"
    stroke-linecap="round" stroke-linejoin="round">
    <!-- Icon paths -->
  </g>
</svg>
```

---

## 8. Logos with Dimension

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 200">
  <defs>
    <linearGradient id="logo-fill" x1="0" y1="0" x2="0.5" y2="1"
      color-interpolation="linearRGB">
      <stop offset="0%" stop-color="#6366f1"/>
      <stop offset="50%" stop-color="#4f46e5"/>
      <stop offset="100%" stop-color="#3730a3"/>
    </linearGradient>
  </defs>
  <!-- Cast shadow -->
  <circle cx="103" cy="106" r="72" fill="#1e1b4b" opacity="0.15"
    filter="url(#soft)"/>
  <!-- Main shape -->
  <circle cx="100" cy="100" r="70" fill="url(#logo-fill)"/>
  <!-- Top highlight arc -->
  <path d="M45,75 Q100,40 155,75" fill="none" stroke="white" stroke-width="2"
    opacity="0.15" stroke-linecap="round"/>
  <!-- Emblem -->
  <path d="..." fill="white"/>
</svg>
```

---

## 9. Animation (CSS + SMIL)

### CSS Animation Essentials
Always pair with:
```css
transform-box: fill-box;
transform-origin: center;
```

### Spinner
```xml
<style>
  .spin {
    fill: none; stroke: #3b82f6; stroke-width: 4;
    stroke-linecap: round; stroke-dasharray: 80 120;
    animation: rotate 1s linear infinite;
    transform-box: fill-box; transform-origin: center;
  }
  @keyframes rotate { to { transform: rotate(360deg); } }
</style>
```

### Line Drawing Reveal (using pathLength)
```xml
<style>
  .draw {
    stroke-dasharray: 1; stroke-dashoffset: 1;
    animation: reveal 2s ease forwards;
    transform-box: fill-box;
  }
  @keyframes reveal { to { stroke-dashoffset: 0; } }
</style>
<path pathLength="1" class="draw" d="..." fill="none" stroke="#333" stroke-width="2"/>
```

### Staggered Entrance
```css
.item {
  opacity: 0; transform: translateY(10px);
  animation: fadeUp 0.5s ease forwards;
  transform-box: fill-box; transform-origin: center;
}
.item:nth-child(1) { animation-delay: 0s; }
.item:nth-child(2) { animation-delay: 0.15s; }
.item:nth-child(3) { animation-delay: 0.3s; }
@keyframes fadeUp {
  to { opacity: 1; transform: translateY(0); }
}
```

### SMIL Animation (Works in `<img>` Tags)
SMIL has 96%+ browser support and works when SVG is used as `<img src>` or CSS `background-image` — where CSS and JS animations do NOT work. Use for self-contained animated SVGs.

**Attribute animation:**
```xml
<circle cx="50" cy="50" r="20" fill="#3b82f6">
  <animate attributeName="r" values="20;25;20" dur="2s" repeatCount="indefinite"/>
</circle>
```

**Transform animation:**
```xml
<rect x="-10" y="-10" width="20" height="20" fill="#ef4444">
  <animateTransform attributeName="transform" type="rotate"
    values="0 50 50;360 50 50" dur="3s" repeatCount="indefinite"/>
</rect>
```

**Motion along a path:**
```xml
<circle r="5" fill="#3b82f6">
  <animateMotion dur="4s" repeatCount="indefinite" rotate="auto">
    <mpath href="#motion-path"/>
  </animateMotion>
</circle>
<path id="motion-path" d="M50,200 Q200,50 350,200" fill="none"/>
```

**Sequential timing:**
```xml
<circle cx="50" cy="50" r="0" fill="#3b82f6">
  <animate id="grow" attributeName="r" from="0" to="20" dur="0.5s" fill="freeze"/>
</circle>
<circle cx="100" cy="50" r="0" fill="#ef4444">
  <animate attributeName="r" from="0" to="20" dur="0.5s" fill="freeze"
    begin="grow.end+0.2s"/>
</circle>
```

### Reduced Motion
Always include in animated SVGs:
```xml
<style>
  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
      animation-duration: 0.01ms !important;
      animation-iteration-count: 1 !important;
      transition-duration: 0.01ms !important;
    }
  }
</style>
```

---

## 10. Data Visualizations

### Bar Chart (Gradient + Shadows)
```xml
<defs>
  <linearGradient id="bar" x1="0" y1="0" x2="0" y2="1" color-interpolation="linearRGB">
    <stop offset="0%" stop-color="#93c5fd"/>
    <stop offset="50%" stop-color="#3b82f6"/>
    <stop offset="100%" stop-color="#1d4ed8"/>
  </linearGradient>
</defs>
<rect x="40" y="80" width="50" height="170" rx="4" fill="url(#bar)"
  filter="url(#drop-shadow)"/>
```

### Donut Chart
```xml
<!-- Shadow -->
<circle cx="200" cy="205" r="80" fill="none" stroke="#1e1b4b" stroke-width="30"
  opacity="0.1" filter="url(#soft)"/>
<!-- Segment: stroke-dasharray = (percentage × circumference), total circumference -->
<circle cx="200" cy="200" r="80" fill="none" stroke="#3b82f6" stroke-width="30"
  stroke-dasharray="201 503" transform="rotate(-90 200 200)" stroke-linecap="round"/>
```

### Axis Lines
Grid: `#f1f5f9`, stroke-width 0.5. Axes: `#94a3b8`, stroke-width 1. Labels: `font-size="11"`, `fill="#64748b"`. Round caps everywhere.

---

## 11. Patterns and Backgrounds

### Mesh Gradient Workaround
Layer 3–5 overlapping radial gradients with different positions, offset focal points, and `spreadMethod="pad"`:
```xml
<defs>
  <radialGradient id="blob1" cx="30%" cy="25%" r="50%" fx="25%" fy="20%"
    color-interpolation="linearRGB">
    <stop offset="0%" stop-color="#818cf8" stop-opacity="0.6"/>
    <stop offset="100%" stop-color="#818cf8" stop-opacity="0"/>
  </radialGradient>
  <radialGradient id="blob2" cx="70%" cy="60%" r="45%"
    color-interpolation="linearRGB">
    <stop offset="0%" stop-color="#f472b6" stop-opacity="0.5"/>
    <stop offset="100%" stop-color="#f472b6" stop-opacity="0"/>
  </radialGradient>
</defs>
<rect width="800" height="600" fill="#0f172a"/>
<rect width="800" height="600" fill="url(#blob1)"/>
<rect width="800" height="600" fill="url(#blob2)"/>
<!-- Add noise overlay for organic feel -->
<rect width="800" height="600" filter="url(#salt-pepper)" opacity="0.06"/>
```

### Pattern Tiles
```xml
<!-- Dots -->
<pattern id="dots" patternUnits="userSpaceOnUse" width="20" height="20">
  <circle cx="10" cy="10" r="1.5" fill="#cbd5e1"/>
</pattern>

<!-- Diagonal lines (use patternTransform) -->
<pattern id="diag" patternUnits="userSpaceOnUse" width="10" height="10"
  patternTransform="rotate(45)">
  <line x1="0" y1="0" x2="0" y2="10" stroke="#e2e8f0" stroke-width="1"/>
</pattern>

<!-- Waves -->
<pattern id="wave" patternUnits="userSpaceOnUse" width="100" height="20">
  <path d="M0,10 Q25,0 50,10 Q75,20 100,10" fill="none" stroke="#e2e8f0"/>
</pattern>
```

---

## 12. Power Features Reference

### `paint-order`
Controls render order. Default: fill → stroke → markers.
`paint-order="stroke fill"` renders stroke behind fill — creates outlined text/halo effects without duplicate elements.

### `vector-effect="non-scaling-stroke"`
Keeps stroke width constant regardless of element transforms/zoom. Essential for technical illustrations and icons at multiple scales.

### `pathLength="1"`
Normalizes path length for dash calculations. `stroke-dasharray="0.5 0.5"` = 50% dashed regardless of actual path length. Simplifies line-drawing animations dramatically.

### `gradientTransform`
Apply transforms to gradient coordinate systems:
```xml
<linearGradient id="angled" gradientTransform="rotate(30)">
```
No need to recalculate x1/y1/x2/y2 — just rotate.

### `spreadMethod`
On gradients — controls behavior beyond boundaries:
- `pad` (default) — extends last color
- `reflect` — mirror effect (great for metallic surfaces)
- `repeat` — tiles the gradient

### Nested `<svg>` Elements
Create independent coordinate systems within parent SVG. Each nested SVG has its own viewBox — useful for components with different aspect ratios or coordinate scales.

### `<symbol>` + `<use>` + CSS Custom Properties
```xml
<symbol id="icon" viewBox="0 0 24 24">
  <circle cx="12" cy="12" r="10" fill="var(--icon-bg, #3b82f6)"/>
  <path d="..." stroke="var(--icon-stroke, white)"/>
</symbol>
<use href="#icon" x="10" y="10" width="48" height="48"
  style="--icon-bg: #ef4444; --icon-stroke: #fff"/>
```

### `color-interpolation: linearRGB`
On gradient elements for smoother, more physically accurate color transitions. Prevents dark banding in gradient midpoints. Default sRGB can make mid-range blends appear darker than expected.

---


## 13. Characters and Figures

### The Fundamental Challenge

LLMs cannot see what they generate. Assembling characters from separate shapes produces gaps, misalignment, and disconnection because the model cannot verify coordinate math visually. This applies to ALL approaches: separate shapes, overlapping shapes, joint covers, and single-path silhouettes.

### Solution: Visual Feedback Loop

The only reliable approach for SVG characters is **iterative refinement with visual feedback**:

1. Write SVG → render to PNG → view the image → identify problems → fix SVG → repeat
2. Build incrementally: torso first → render → add legs → render → add arms → render
3. Expect 4-8 iterations for a good character

See the main SKILL.md for the complete render-and-view workflow.

### Static Characters: Thick Rounded Lines

The simplest approach that produces connected bodies. Use `<line>` with `stroke-linecap="round"` and large `stroke-width`. Round end caps create natural tapered limb shapes. Add `<circle>` at every joint drawn AFTER the lines.

```xml
<!-- Torso -->
<line x1="200" y1="180" x2="200" y2="300" stroke="#1e40af" stroke-width="30" stroke-linecap="round"/>
<!-- Upper arm -->
<line x1="200" y1="195" x2="160" y2="260" stroke="#dea87a" stroke-width="16" stroke-linecap="round"/>
<!-- Shoulder joint cover (on top) -->
<circle cx="200" cy="195" r="14" fill="#dea87a"/>
```

Always render and verify each piece after adding it.

### Animated Characters: React + Forward Kinematics

For animated characters, use a **React/JSX artifact** with mathematical position computation. This guarantees connected joints because every position is computed from its parent:

```jsx
// Forward kinematics: parent position + angle + bone length → child position
function fk(px, py, angle, length) {
  return {
    x: px + Math.cos(angle) * length,
    y: py + Math.sin(angle) * length,
  };
}

// Define skeleton
const BONES = { upperArm: 50, forearm: 45, thigh: 60, shin: 55 };

// Compute all positions from angles (animated via useState/useEffect)
const shoulder = { x: 200, y: 150 };
const elbow = fk(shoulder.x, shoulder.y, shoulderAngle, BONES.upperArm);
const wrist = fk(elbow.x, elbow.y, shoulderAngle + elbowAngle, BONES.forearm);
// elbow is ALWAYS exactly 50px from shoulder. wrist ALWAYS 45px from elbow.

// Render as SVG inside JSX
<line x1={shoulder.x} y1={shoulder.y} x2={elbow.x} y2={elbow.y}
  stroke="#dea87a" strokeWidth="16" strokeLinecap="round"/>
```

Key principles:
- Define **bone lengths** (constants) and **joint angles** (animated values)
- Use `fk()` to compute EVERY joint position from its parent
- Animate by interpolating ANGLES, not positions
- Render with `<line>` + `strokeLinecap="round"` + `<circle>` joint covers

### Proportions (8-Head System)

Standing adult: total height = 8 × head height. Shoulders ≈ 2.5 head-widths. Hips ≈ 1.5. Elbow at waist. Wrist at crotch. Knee at 2 heads from ground.

### Animation Timing

- Walk: 1–1.2s. Run: 0.5–0.7s. Push-up: 2–3s. Jump: 1.5s. Breathing: 3–4s.
- Use `ease-in-out` for organic motion.

### When to Recommend External Tools

For **production-quality** character animation, recommend:
- **Rive** (rive.app) — state machine-based, runs on web/iOS/Android/Flutter
- **Lottie** — After Effects → JSON, plays via lottie-web
- **Spine** — game-focused 2D skeletal animation

Offer to generate the static SVG character as a starting point for import into these tools.
