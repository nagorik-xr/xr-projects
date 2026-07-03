# v1 — MindAR engine (frozen 2026-07-03)

Sunglasses virtual try-on, WebAR, single-file app. Kept for comparison against v2 (Jeeliz VTO).

## Engine
- **Tracking:** MindAR 1.2.5 face tracking (`mindar-face-aframe.prod.js`, CDN) — wraps Google MediaPipe FaceMesh (468 landmarks).
- **Rendering:** A-Frame 1.4.2 (three.js r147).
- **No build step.** Open `index.html` over any static HTTPS/localhost server.

## How it was built, step by step
1. Copied proven anchor/scale values from MindAR's official `examples/face-tracking/example2.html`.
2. Glasses anchored at face landmark **anchorIndex 168** (nose bridge) via `mindar-face-target`.
3. Head occluder (`sparkar/headOccluder.glb`, scale 0.065) at same anchor — hides temple arms behind head. Approximate proxy mesh, not real ear detection.
4. Assets: Khronos `SunglassesKhronos.glb` (aviator) + MindAR example `glasses2` (wayfarer, Spark AR sample).
5. **Gotcha:** MindAR injects camera `<video>` at `z-index:-2` — any opaque background on the container hides the feed. Container must stay transparent.
6. **Gotcha:** A-Frame 1.4's three.js ignores `KHR_materials_transmission` → aviator lenses rendered invisible. Fixed with `dark-lenses` component: replaces `lens_exterior` material with dark `MeshStandardMaterial`, hides `lens_interior`.
7. Fit calibrated headless (playwright + canvas-stream fake webcam feeding a portrait photo — see scratchpad probe.js technique). Base: aviator scale 6.5 pos `0 -0.32 0`; wayfarer scale 0.42 rotY -90.
8. **Auto-fit:** reads `scene.systems["mindar-face-system"].controller.lastEstimateResult.metricLandmarks`, temple landmarks 234/454 distance, median of 15 frames ÷ `FACE_W_REF 15.45` → global scale multiplier. Runs on first `targetFound` + manual button.
9. Fit panel: Size/Height/Depth/Tilt sliders, stored per style in localStorage `tryon-fit-v2`.
10. Smoothing: One-Euro filter `filterMinCF: 0.0005, filterBeta: 0.5` (defaults 0.001/1).

## Verdict (why frozen)
Fit ±10%, edge-landmark jitter, no real ear/nose occlusion — MediaPipe FaceMesh ceiling, not fixable by tuning. Switching to Jeeliz VTO (purpose-built eyewear tracker).
