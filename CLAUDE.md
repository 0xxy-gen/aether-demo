# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Aether Space — Mission Manifest Optimizer (MMO-2026-0031)**

A self-contained, single-file interactive 3D web application for space launch vehicle payload manifest optimization. Operators use it to configure payloads on rideshare missions (Falcon 9, Electron, SSLV) and run a simulated constraint solver.

## Running the App

No build step. Open directly in a WebGL-capable browser:

```bash
open index.html
# or serve locally to avoid any CORS issues:
python3 -m http.server 8080
```

Three.js r128 is loaded from CDN — an internet connection is required on first load.

## Architecture

Everything lives in **`index.html`** (~2,100 lines). The file is structured as:

1. **HTML** — 3-panel layout: left (mission params), center (Three.js viewport), right (launch options). Includes a scrubber/timeline and a trace-log drawer.
2. **CSS** — Dark aerospace theme using CSS custom properties (`--bg`, `--surface`, `--acc`, `--warn`, etc.; IBM Plex Mono/Sans fonts, cyan/gold palette).
3. **JavaScript** — Vanilla ES6+, no framework.

### State

All mutable UI and solver state lives in a single object `S`:

```js
const S = {
  mass, alt, inc, iface,           // mission parameters
  envL, envW, envH,                // payload envelope (mm)
  view, solved, showFairing,
  showAdapters, showMass, selOpt,  // UI mode
  pbPhase, pbProgress, pbPlaying,
  pbSpeed, pbStartTime, pbAnimId,
  pbCandidates, pbSurvivors,
  pbOptIdx, swapAngle,
  shuffleSchedule,                 // payload box shuffle events
}
```

### Key Functions

| Function | Purpose |
|---|---|
| `runSolve()` | `async` — kicks off solver simulation; populates `pbCandidates` and begins playback |
| `buildFairingScene(vIdx)` | Tears down and rebuilds the entire Three.js scene for the selected vehicle |
| `buildCandidateMeshes(candidates)` | Creates ghost payload meshes for solver animation |
| `buildSolverTrace()` | Generates trace-log entries shown in the drawer |
| `buildShuffleSchedule()` | Pre-computes payload box shuffle events for solver animation |
| `updateCandidateOpacities()` | Animates payload candidate states during scrubber playback |
| `updateUserPayloadSwap(dt)` | Animates user payload position between ESPA ring ports during solve |
| `renderLoop()` | `requestAnimationFrame` loop — updates camera, solver playback, and renders |
| `selectOpt(i)` | Switches active launch vehicle and rebuilds scene |
| `renderResults()` | Populates right panel with ranked launch option cards |
| `fairingRadiusAtWorldY(worldY)` | Returns fairing inner radius at a given Y height (for fit checks) |
| `checkPayloadFit()` | Validates payload envelope against selected vehicle fairing |
| `updateEnvelope()` | Syncs envelope sliders/inputs and triggers fit check |

### 3D Scene

- **Camera:** spherical coordinates (theta/phi/distance), 3 preset views (ISO, Top, Side), mouse/touch orbit + pinch-to-zoom
- **Geometry:** procedurally generated — true tangent-ogive fairing profile (dimensions from Falcon 9 IGS file), solid ESPA ring torus, wireframe payload boxes via `mkBoxWireframe()`
- **Materials:** pure wireframe lines for fairing; `MeshStandardMaterial` + emissive for ESPA ring and payload boxes; cyan for user payload, blue for manifest payloads, gold for ESPA ring
- **Scene groups:** `fairingGroup`, `payloadGroup`, `espaGroup`, `massGroup`, `mountGroup`, `primaryPayloadGrp`, `cakeGhostGrp`, `adapterGrpRef`
- **Cached refs:** `userPayloadBodyMat`, `payloadGroupGroups` — updated after each `buildFairingScene()` to avoid per-frame traversal

### Solver Simulation

5 phases: Seeding → Placement → Constraint Sweep → Convergence → Solution Lock. Runs over a fixed 8-second window (`PB_DURATION = 8000`). Speed is adjustable (0.5×–4×).

### Launch Vehicle Data

Hardcoded in the `VEHICLES` array — **Falcon 9 Block 5** (SpaceX), **Neutron Maiden Flight** (Rocket Lab), **SSLV Rideshare** (ISRO) — each with fairing dimensions (`fairR`, `fairH`), ESPA ring radius (`espaR`), port count (`nPorts`), payload slot manifest, orbital characteristics, cost/risk metrics, and a `color` for scene theming.

## Git Workflow

### Branching

- `main` — stable, always deployable (open directly in browser)
- Feature branches: `feature/<short-description>` (e.g. `feature/fairing-lod`)
- Bug fixes: `fix/<short-description>` (e.g. `fix/scrubber-drift`)

### Commit Style

Use short imperative subject lines (≤ 72 chars). Reference the subsystem in scope:

```
feat(solver): add phase 6 convergence tightening
fix(scene): correct ogive profile radius calculation
style(css): update cyan token to match brand palette
docs(claude): add git workflow section
```

Common scopes: `solver`, `scene`, `ui`, `css`, `data`, `docs`, `claude`.

### Pull Request Process

1. Branch off `main`, keep PRs focused on one concern.
2. Test by opening `index.html` directly in a WebGL browser.
3. Verify all three vehicles render and solver completes without console errors.
4. Squash-merge into `main` with a descriptive commit message.

### What Not to Commit

- Editor/IDE config files (`.vscode/`, `.idea/`)
- OS artifacts (`.DS_Store`, `Thumbs.db`)
- Local server logs or generated output files
