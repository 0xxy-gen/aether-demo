# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Aether Space — Mission Manifest Optimizer (MMO-2026-0031)**

A self-contained, single-file interactive 3D web application for space launch vehicle payload manifest optimization. Operators use it to configure payloads on rideshare missions (Falcon 9, Electron, SSLV) and run a simulated constraint solver.

## Running the App

No build step. Open directly in a WebGL-capable browser:

```bash
open aether_space_mmo_v7.html
# or serve locally to avoid any CORS issues:
python3 -m http.server 8080
```

Three.js r128 is loaded from CDN — an internet connection is required on first load.

## Architecture

Everything lives in **`aether_space_mmo_v7.html`** (~1,420 lines). The file is structured as:

1. **HTML** — 3-panel layout: left (mission params), center (Three.js viewport), right (launch options). Includes a scrubber/timeline and a trace-log drawer.
2. **CSS** — Dark aerospace theme using CSS custom properties (`--c-*` tokens, IBM Plex Mono/Sans fonts, cyan/gold palette).
3. **JavaScript** — Vanilla ES6+, no framework.

### State

All mutable UI and solver state lives in a single object `S`:

```js
const S = {
  mass, alt, inc,           // mission parameters
  view, solved,             // UI mode
  pbPhase, pbProgress,      // playback position
  pbCandidates, pbSurvivors // solver candidates
}
```

### Key Functions

| Function | Purpose |
|---|---|
| `runSolve()` | Kicks off the solver simulation; populates `pbCandidates` and begins playback |
| `buildFairingScene(vIdx)` | Tears down and rebuilds the entire Three.js scene for the selected vehicle |
| `buildSolverTrace()` | Generates the trace-log entries shown in the drawer |
| `updateCandidateOpacities()` | Animates payload candidate states during scrubber playback |
| `renderLoop()` | `requestAnimationFrame` loop — updates camera, solver playback, and renders |
| `selectOpt(i)` | Switches active launch vehicle and rebuilds scene |

### 3D Scene

- **Camera:** spherical coordinates (theta/phi/distance), 3 preset views (iso, top, side), mouse/touch orbit
- **Geometry:** procedurally generated — true ogive fairing profile, ESPA ring with port holes punched via subtraction geometry, semi-transparent payload boxes
- **Materials:** wireframe + `MeshStandardMaterial` with emissive for glow; material state (eliminated / survivor / optimal) swapped during playback

### Solver Simulation

5 phases: Seeding → Placement → Constraint Sweep → Convergence → Solution Lock. Runs over a fixed 8-second window (`PB_DURATION = 8000`). Speed is adjustable (0.5×–4×).

### Launch Vehicle Data

Hardcoded in JS — Falcon 9 Block 5, Electron Kick Stage, SSLV Rideshare — each with fairing dimensions, payload ports, orbital characteristics, and cost/risk metrics.

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
2. Test by opening `aether_space_mmo_v7.html` directly in a WebGL browser.
3. Verify all three vehicles render and solver completes without console errors.
4. Squash-merge into `main` with a descriptive commit message.

### What Not to Commit

- Editor/IDE config files (`.vscode/`, `.idea/`)
- OS artifacts (`.DS_Store`, `Thumbs.db`)
- Local server logs or generated output files
