# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A classic Tetris implementation in vanilla JavaScript with HTML5 Canvas. No dependencies, no build step, no package.json — just static files.

## Running the game

There is no build/lint/test tooling. To run:

```bash
# Open directly
start index.html          # Windows

# Or serve locally (recommended, avoids any file:// quirks)
python3 -m http.server 8000
# or
npx serve .
```

Then open `http://localhost:8000`. Changes to `game.js`/`style.css`/`index.html` take effect on browser refresh — no compile/bundle step.

There are no automated tests. Verify changes by playing the game in a browser (use the `run` skill or manually open `index.html`).

## Architecture

Three files, ~300 lines total, all logic lives in `game.js`:

- **`index.html`** — DOM structure: main `<canvas id="board">` (300×600, 10×20 grid of 30px blocks), a `<canvas id="next-canvas">` for the next-piece preview, HUD elements (`#score`, `#lines`, `#level`), and a shared `#overlay` used for both pause and game-over states.
- **`style.css`** — dark/retro arcade visual theme only; no layout logic depends on it.
- **`game.js`** — entire game engine, single file, no modules:
  - **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
  - **Pieces**: `PIECES` are square matrices (see top of file). Rotation is computed on the fly via `rotateCW` (transpose + reverse), not stored as precomputed rotation states.
  - **Collision** (`collide`): checks board bounds and existing locked cells.
  - **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` until one doesn't collide, else the rotation is discarded.
  - **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and advances the piece when it exceeds `dropInterval`.
  - **Line clearing** (`clearLines`): scans bottom-up, splices full rows out and unshifts empty rows at the top.
  - **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/cell dropped, soft drop adds 1 pt/row.
  - **Leveling/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
  - **Ghost piece**: `ghostY()` projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2`.
  - All rendering goes through `drawBlock`/`draw`/`drawNext` — no other rendering paths exist.
  - Game state (`board`, `current`, `next`, `score`, etc.) is module-level mutable state, not encapsulated in a class/object. `init()` resets all of it and (re)starts the RAF loop; it's also the restart handler.

## Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `PIECES`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `width`/`height` attributes of `<canvas id="board">` in `index.html` to match (`COLS×BLOCK` by `ROWS×BLOCK`).
