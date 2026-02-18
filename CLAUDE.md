# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**wigglydoom** is a static, zero-build interactive 3D particle visualization. No npm, no bundler, no build step — open `index.html` directly in a browser.

## Running the Project

Open `index.html` in a browser (it redirects to `globe.html` with cache-busting). Or serve locally with any static server:

```bash
npx serve .
# or
python3 -m http.server 8080
```

No dependencies to install. Three.js v0.153.0 loads from CDN.

## Architecture

The entire app lives in `globe.html` (a single self-contained HTML file with inline JS). `index.html` is just a redirect.

**Two script blocks in `globe.html`:**

1. **Main visualization** (runs immediately):
   - Sets up Three.js `Scene`, `PerspectiveCamera`, `WebGLRenderer`
   - `createGeometry(shape, size, density)` — generates point cloud geometries for 6 shapes: `sphere`, `randomSphere`, `cube`, `pyramid`, `cylinder`, `face`
   - `customShaderMaterial` — a `THREE.ShaderMaterial` with custom GLSL shaders
   - Vertex shader: animates particles with wave/ripple displacement, pulse effects, and a special voice-mode mouth animation
   - Fragment shader: 4 color modes (Rainbow, Pink/Purple, Gold/Blue, Cyan/Lime) using cosine palette functions
   - Animation loop handles rotation, mouse-drag inertia, and voice input processing
   - All settings persist via URL query parameters

2. **Voice control** (runs on `window.load`):
   - Web Audio API (`AudioContext`, `AnalyserNode`, `getUserMedia`)
   - Mic volume maps to `rippleIntensity` and `rippleSpeed` shader uniforms via LERP smoothing (`smoothingFactor = 0.15`)
   - `voiceMode` uniform restricts wiggling to mouth vertices when `face` shape is active

**Key shader uniforms** (JS → GLSL):
```
time, colorMode, rippleSpeed, rippleIntensity,
pulseTime, pulseCenter, gradientStart, gradientEnd,
voiceMode, mouthYPosition
```

## Key Files

- [globe.html](globe.html) — the entire application
- [globe_original.html](globe_original.html) — backup before voice control and face shape were added
- [style.css](style.css) — global styles (UI controls, layout)
- [index.html](index.html) — redirect entry point with cache-busting
