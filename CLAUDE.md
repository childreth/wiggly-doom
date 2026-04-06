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

## Additional Experiences

Beyond `globe.html`, the repo contains two standalone HTML pages that share `style.css` but are otherwise self-contained:

- **`not-the-lawnmowerman.html`** — webcam-based face-tracking experience. Uses a real-time ML model (MediaPipe or similar) running on-device to detect facial landmarks via `getUserMedia`. Has a "bite game" mechanic that plays `sounds/bite.mp3` and `sounds/gameover.mp3`. Uses `style-ntlm.css` for its extra styles.
- **`hand-banana.html`** — another webcam/gesture experience under the wigglydoom umbrella. Shares the same UI patterns (status bar, backend badge, start/drop buttons).

Both pages link back to `index.html` and rely on GPU/WebGL acceleration (displayed via `#backend-badge`).

## Key Files

- [globe.html](globe.html) — the entire main application
- [globe_original.html](globe_original.html) — backup before voice control and face shape were added
- [not-the-lawnmowerman.html](not-the-lawnmowerman.html) — webcam face-tracking + bite game
- [hand-banana.html](hand-banana.html) — webcam gesture experience
- [style.css](style.css) — shared global styles (UI controls, layout)
- [style-ntlm.css](style-ntlm.css) — extra styles specific to not-the-lawnmowerman
- [index.html](index.html) — redirect entry point with cache-busting
- [sounds/](sounds/) — `bite.mp3`, `gameover.mp3` used by the bite game
