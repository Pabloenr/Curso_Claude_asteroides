# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas + vanilla JS. No dependencies, no bundler, no tests, no linter, and not a git repo. The README and in-game text are in Spanish; keep new user-facing strings and comments in Spanish for consistency.

## Running

Open `index.html` directly in a browser, or serve it with `npx serve .` (http://localhost:3000). There is no build step; edit and reload.

## Architecture

All logic is in `game.js` (loaded by `index.html` via a plain `<script>`, strict mode). `index.html` only provides the 800×600 canvas (`W`/`H` constants in `game.js` must match its `width`/`height`).

- **Entity classes** (`Ship`, `Bullet`, `Asteroid`, `Particle`) each have `update(dt)` and `draw()`, and draw directly on the global `ctx`. Entities are marked with `dead = true` and removed by `filter` in `update()`, never spliced mid-iteration.
- **Global game state** (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`) lives in module-level `let`s, reset by `initGame()`. `state` is `'playing' | 'dead' | 'gameover'`; `'dead'` is a 2s respawn timer (`deadTimer`) during which the world keeps simulating except ship/bullets.
- **Loop**: `loop(ts)` → `update(dt)` → `draw()`; `dt` is in seconds and clamped to 0.05 to avoid tunneling after tab switches.
- **World is toroidal**: positions wrap with `wrap()`. Collisions use plain center distance (`dist`) against `radius`, with no wrap-aware distance, so objects straddling an edge don't collide across it.
- **Input**: `keys[code]` is held state; `pressed(code)` is an edge-triggered "just pressed" that is consumed on read (used for Space to shoot and to restart). Arrow keys/Space call `preventDefault`.
- **Asteroid sizes** are 1–3 and index the parallel arrays `RADII`, `SPEEDS`, `POINTS` (index 0 unused). Size 3 = large (20 pts), size 1 = small (100 pts). `split()` produces two asteroids of `size - 1`. Adding a size or tuning values means editing all three arrays.
- **Levels**: clearing all asteroids calls `nextLevel()`, which spawns `3 + level` large asteroids outside a 130px safe radius around the center and resets the ship (3s invincibility with blinking).

Note: the README mentions power-ups and a "shooting star" asteroid type, but neither exists in the code yet.
