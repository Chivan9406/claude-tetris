# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step or dependencies. Open directly in a browser:

```bash
start index.html          # Windows
open index.html           # macOS
```

Or serve locally (avoids file:// quirks):

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Architecture

Three files, no framework, no bundler:

- **`index.html`** — DOM structure: a `<canvas id="board">` (300×600 px) for the playfield and a `<canvas id="next-canvas">` (120×120 px) for the next-piece preview, plus a `#overlay` div for PAUSE / GAME OVER states.
- **`style.css`** — dark/retro arcade theme; overlay uses `backdrop-filter`.
- **`game.js`** — all game logic (~300 lines, `'use strict'`, no modules).

### Key data model (`game.js`)

- `board` — `ROWS × COLS` (20×10) array; cells hold `0` (empty) or a color index 1–7.
- `current` / `next` — piece objects `{ type, shape, x, y }` where `shape` is a 2-D array of color indices.
- `PIECES` — piece definitions indexed 1–7 (I, O, T, S, Z, J, L); index 0 is `null`.
- `COLORS` — parallel color palette; index 0 is `null`.

### Game loop

`init()` → `spawn()` → `requestAnimationFrame(loop)`. Each `loop` tick accumulates elapsed time in `dropAccum`; when it exceeds `dropInterval` the piece falls one row or locks. `draw()` repaints the full canvas every frame.

### Core functions

| Function | Role |
|---|---|
| `collide(shape, ox, oy)` | Bounds + overlap check |
| `rotateCW(shape)` | Transpose + reverse rows |
| `tryRotate()` | Rotation with ±1/±2 column wall kicks |
| `lockPiece()` | `merge()` → `clearLines()` → `spawn()` |
| `ghostY()` | Projects drop position for ghost piece |
| `clearLines()` | Splices full rows and updates score/level/speed |

### Tunable constants

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES` are all at the top of `game.js`. If you change `COLS`, `ROWS`, or `BLOCK`, update the canvas `width`/`height` attributes in `index.html` accordingly (`COLS × BLOCK` and `ROWS × BLOCK`).