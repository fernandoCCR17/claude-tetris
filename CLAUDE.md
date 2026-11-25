# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris (HTML5 Canvas + CSS). No dependencies, no build step, no package.json, no tests, no linter. The README and UI text are in Spanish; keep user-facing strings in Spanish.

## Running

Open `index.html` directly, or serve statically (e.g. `python -m http.server 8000`, then http://localhost:8000).

## Architecture

Three files, all logic in `game.js` (a classic script loaded by `index.html`, not an ES module, `'use strict'`):

- `index.html` defines the DOM that `game.js` looks up by id at load time (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `resume-btn`, `restart-btn`). Renaming an id requires updating the lookups at the top of `game.js`.
- `game.js` keeps all state in module-level `let` globals (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, ...), reset by `init()`, which is also the restart handler.
- Board is a `ROWS×COLS` (20×10) matrix of ints; `0` is empty and `1..7` index into both `COLORS` and `PIECES` (I, O, T, S, Z, J, L). Pieces are `{type, shape, x, y}` where `shape` is a matrix holding its own color index, so merging copies cell values straight into the board.

Game flow: `requestAnimationFrame` `loop` accumulates `dropAccum` and drops the piece when it reaches `dropInterval`. A blocked drop calls `lockPiece()`, which runs `merge()`, `clearLines()` and `spawn()`. `spawn()` promotes `next`, generates a new one and calls `endGame()` if the spawn position collides. `endGame()` and pause both cancel the animation frame; `togglePause()` and `init()` restart the loop manually, so any new code path that stops or starts the loop must manage `animId` and `lastTime` itself.

Gotchas:
- `collide()` reads the global `board`, and treats `ny < 0` as free (pieces can sit above the top).
- `clearLines()` owns the score, level and `dropInterval` formulas (`level = floor(lines/10)+1`, `dropInterval = max(100, 1000 - (level-1)*90)`).
- Soft drop (+1/cell) and hard drop (+2/cell) score in `softDrop()`/`hardDrop()`. The gravity step inside `loop` does not score.
- Rotation (`tryRotate`) is clockwise only, with horizontal wall kicks `[0,-1,1,-2,2]`.
- Rendering is split: `draw()` is called every frame (board, ghost, current piece). `drawNext()` is only called from `spawn()`. `updateHUD()` must be called explicitly after score, line or level changes.
- Canvas dimensions are hardcoded in `index.html` (300×600 = `COLS*BLOCK` × `ROWS*BLOCK`), so changing `COLS`, `ROWS` or `BLOCK` in `game.js` also requires updating them there and in `style.css`.
