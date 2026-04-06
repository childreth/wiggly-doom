# not-the-lawnmowerman.html

Webcam face-tracking game built on MediaPipe + Three.js. Single self-contained HTML file, no build step.

## What it does

Uses **MediaPipe FaceLandmarker** (478 landmarks, WebGPU delegate with CPU fallback) to track the user's face in real time. The landmarks are rendered as an animated Three.js point cloud using a custom GLSL wiggle shader (adapted from `globe.html`).

On top of the face visualization, there's a **bite game** with two mechanics:

- **Bite** — open your mouth wide (jaw threshold `0.38`) to eat food that overlaps your mouth region
- **Laser** — blink either eye (blink threshold `0.45`) to fire dual red laser beams from your eyes; 30-frame cooldown between shots

Food objects (apple, donut, burger, pizza, strawberry, avocado, lemon, ice cream) fly horizontally across the screen. Missing food costs a life. 3 lives per game. Score increases with difficulty (spawn interval decreases, speed increases over time).

## Architecture

### Detection
- `FaceLandmarker` runs in `VIDEO` mode, decoupled from the render loop via `detectionLoop()` + `requestAnimationFrame`
- Blendshapes (`outputFaceBlendshapes: true`) drive expression colors and game inputs:
  - `jawOpen` → bite mechanic + mouth bounding box height
  - `eyeBlinkLeft` / `eyeBlinkRight` → laser trigger
  - `mouthSmileLeft/Right`, `browDownLeft/Right`, `browInnerUp` → expression color overlay (green/red/yellow)

### Geometry
- 478 base landmarks pre-allocated up to `MAX_LANDMARKS = 4780` (10× density)
- `dotDensity > 1` uses **barycentric supersampling** on the face mesh tesselation — extra dots are stable (pre-baked `randBuf`) and track face movement
- `dotDensity < 1` subsamples by stride

### Laser system
- Face normal computed from forehead/chin/cheek landmarks → `aimDir` points into screen
- Two beams spawn from `leftEyeWorld` / `rightEyeWorld`, angled outward (`OUTWARD_ANGLE = 0.22`)
- Hit detection: cone-spread raycasting (9 rays per beam, `SPREAD_ANGLE = 0.20` rad) + proximity fallback (`PROX_RADIUS = 0.8` world units)
- Beams are `CylinderGeometry` cones (wide far end) with glow layer; auto-removed after `LASER_VISIBLE_MS = 150ms`

### Shader uniforms (JS → GLSL)
| Uniform | Type | Purpose |
|---|---|---|
| `time` | float | Animation clock |
| `colorMode` | int | 0=rainbow, 1=pink/purple, 2=gold/blue, 3=cyan/lime |
| `rippleSpeed` | float | Wave animation speed |
| `rippleIntensity` | float | Wave displacement amplitude |
| `gradientStart/End` | float | Y-axis gradient band |
| `pointSize` | float | Dot size in pixels |
| `expressionColor` | vec3 | Expression-driven color tint |
| `expressionStrength` | float | Blend weight for expression color |

## Key constants
```js
JAW_THRESHOLD   = 0.38   // mouth open score for bite
BLINK_THRESHOLD = 0.45   // eye blink score for laser
LASER_COOLDOWN  = 30     // frames (~0.5s at 60fps)
LASER_BEAM_LEN  = 28     // world units
SPREAD_ANGLE    = 0.20   // radians, raycaster cone half-angle
PROX_RADIUS     = 0.8    // world units, proximity fallback radius
GRAVITY         = 0.0007 // for "drop dots" animation
```

## Files
- `not-the-lawnmowerman.html` — entire app
- `style.css` — shared UI styles
- `style-ntlm.css` — page-specific styles (HUD, overlays, laser bar, flash effects)
- `sounds/bite.mp3` — bite SFX
- `sounds/gameover.mp3` — game over SFX

## Dependencies (CDN, no install)
- `three@0.153.0` — UMD build, loaded as global `THREE`
- `@mediapipe/tasks-vision@0.10.3` — ES module import for `FaceLandmarker` + `FilesetResolver`
- MediaPipe WASM + model served from Google Storage / jsDelivr CDN
