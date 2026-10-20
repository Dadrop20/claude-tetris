# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No build, no deps, no package.json, no tests, no linter. README and UI text are in Spanish.

Run: open `index.html` directly, or serve statically (`python -m http.server 8000`) and visit `http://localhost:8000`.

## Architecture

- `index.html` — DOM + two canvases: `#board` (300×600 = `COLS*BLOCK` × `ROWS*BLOCK`) and `#next-canvas` (120×120), HUD spans, `#overlay` (pause/game over). Changing `COLS`/`ROWS`/`BLOCK` in `game.js` requires updating the `#board` width/height attrs here.
- `style.css` — dark/retro styling; overlay visibility toggled via `.hidden` class.
- `game.js` — all logic, single script with global mutable state (`board, current, next, score, lines, level, paused, gameOver, ...`) set in `init()`.

Key points in `game.js`:
- Board is `ROWS×COLS` matrix; cell = 0 or color/piece index 1–7 (indexes `COLORS` and `PIECES`, which must stay aligned; index 0 is `null`).
- Pieces are square matrices; `rotateCW` = transpose + reverse. `tryRotate` kicks horizontally only (`[0,-1,1,-2,2]`), no SRS.
- Flow: `loop` (rAF, accumulates `dropAccum` vs `dropInterval`) → `lockPiece` = `merge` → `clearLines` → `spawn`. `spawn` ends the game if the new piece collides.
- `clearLines` owns scoring/level/`dropInterval` (`max(100, 1000-(level-1)*90)`); soft drop +1/row, hard drop +2/row are applied in `softDrop`/`hardDrop`.
- Pause/game over stop the loop via `cancelAnimationFrame(animId)`; `togglePause` and `init` restart it. Any new state that must reset on restart goes in `init()`.
