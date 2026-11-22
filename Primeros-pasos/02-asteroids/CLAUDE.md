# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in vanilla HTML5 Canvas + JavaScript. No dependencies, no bundler, no package.json, no tests, no linter. The README and in-game text are in Spanish.

## Running

Open `index.html` directly in a browser, or serve the folder: `npx serve .` (then http://localhost:3000).

## Architecture

All game logic lives in a single script, `game.js`, loaded by `index.html` onto an 800x600 `<canvas id="canvas">`. `W`/`H` constants in `game.js` must stay in sync with the canvas size in `index.html`.

- **Entities** are classes (`Ship`, `Asteroid`, `Bullet`, `Particle`) with `update(dt)` and `draw()`. Entities that expire set `dead = true` and are removed with array filters in `update()`.
- **Global mutable state** (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`) is reset by `initGame()`. `state` is one of `'playing' | 'dead' | 'gameover'` and drives which branch of `update()` runs.
- **Loop**: `requestAnimationFrame` → `update(dt)` then `draw()`. `dt` is in seconds and clamped to 0.05.
- **World is toroidal**: positions use `wrap()`; there are no edge collisions.
- **Input**: `keys` holds held state; `pressed(code)` is a one-shot edge-triggered check (consumes `justPressed`). Use `keys[...]` for continuous actions (rotate/thrust) and `pressed(...)` for discrete ones (shoot, restart).
- **Collisions** are circle-based using `dist()` against `radius`. Ship-vs-asteroid uses `a.radius * 0.82` as forgiveness. Asteroid size tables `RADII`, `SPEEDS`, `POINTS` are indexed by size 1–3 (index 0 unused).
- **Rendering** is wireframe (white strokes on black); no sprites/assets besides `favicon.svg`.

## Note

README mentions "shooting star" asteroids, but they are not implemented in `game.js`. Three power-ups are implemented via `PowerUp` (`type` = 'triple' | 'shield' | 'nova'; on-screen items in `powerUps`). Spawn odds and per-level caps live in `POWERUP_RULES` (`spawnCounts` tracks appearances; triple/shield up to 3 per level, nova once). Triple/shield use timers (`tripleShotTimer`/`shieldTimer`, shown in the HUD with `drawTimerBar`); nova has no state: picking it up calls `detonateNova()` immediately.
