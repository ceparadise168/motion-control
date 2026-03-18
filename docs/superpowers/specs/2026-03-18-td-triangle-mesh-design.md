# TouchDesigner 風格首頁背景特效

## Summary

Replace the current floating geometry (magnetic line segments) on the start screen with a TouchDesigner-inspired sparse triangle mesh effect with color wave cycling.

## Visual Design

### Triangle Mesh
- Equilateral triangle grid with ~80px spacing (sparse, not dense)
- Vertices displaced by 2D Perlin noise (±35px), creating organic wave-like deformation
- Noise evolves over time at scale 0.003, speed 0.3

### Color Palette (two-tone only)
- **Bright edges/vertices**: Cyan `rgb(0, 255, 255)` or Purple `rgb(160, 100, 255)`
- **Dark edges**: `rgb(80, 90, 110)` — always subtle
- Brightness per edge determined by noise field (threshold 0.5): bright edges get alpha 0.12–0.55, dark edges get alpha 0.02–0.06
- Bright vertices get a 1.8px core dot + 5px glow halo

### Color Wave Cycling
- Cyan ↔ Purple transition via radial wave expanding from a source point
- **Cycle period**: 5 seconds per color switch (10 seconds full round-trip)
- **Wave speed**: 350px/s expansion
- **Transition band**: 80px smooth gradient between old and new color
- **Source point**: orbits screen center at radius `min(W,H) * 0.2`, slow angular velocity
- **Start offset**: time begins at t=3.2s so first frame shows wave mid-spread (not static)

### Particles
- Emitted from bright mesh vertices (noise > 0.5), spawn rate ~12% per frame
- Max 180 particles on screen
- Size: 0.6–1.4px, lifetime: 2–5 seconds
- Move along noise flow field at 0.6px/frame
- Fade out over last 1.2 seconds of life
- Color matches the local wave color at spawn time

## Technical Spec

### What to replace
- Remove the entire `(function() { ... })()` IIFE block for "FLOATING GEOMETRY" (~240 lines, index.html lines 852–1093)
- Keep the `<canvas id="geoCanvas">` element and its CSS unchanged
- Keep the `geoCanvasStop` / `geoCanvasStart` global functions with same API

### Implementation constraints
- Pure Canvas 2D — no WebGL, no external libraries
- Self-contained noise function (same permutation-table approach as prototype)
- Must respect `prefers-reduced-motion` media query
- Must stop animation when game starts (`geoCanvasStop()`) and resume if returning to start screen (`geoCanvasStart()`)
- Use `devicePixelRatio` (capped at 2) for sharp rendering on retina displays

### Performance considerations
- Triangle grid rebuilds on resize only, not every frame
- Edge count scales with screen size (~500–800 edges on typical screens)
- Particle cap of 180 prevents runaway allocation
- Single `requestAnimationFrame` loop, no secondary timers
