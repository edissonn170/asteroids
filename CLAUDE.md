# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step, no dependencies. Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

Then visit `http://localhost:3000`. There are no tests, no linter, no package.json.

## Architecture

The entire game is a single file: `game.js` (~420 lines), loaded by `index.html` into an 800×600 `<canvas>`.

**Game loop** — `requestAnimationFrame` calls `loop(ts)`, which computes a capped delta-time (`dt`, max 50ms) and calls `update(dt)` then `draw()` every frame.

**State machine** — `state` variable drives behavior in `update`:
- `'playing'` — normal gameplay
- `'dead'` — ship just died, 2-second respawn timer, asteroids still move
- `'gameover'` — all lives lost, Space restarts via `initGame()`

**Entity classes** — `Ship`, `Asteroid`, `Bullet`, `Particle` all follow the same pattern: constructor sets initial state, `update(dt)` mutates position/timers, `draw()` renders to `ctx`. Dead entities are filtered from arrays after each frame.

**Collision** — checked in `update` via the `dist()` utility (Euclidean distance). Bullet–asteroid and ship–asteroid pairs are iterated manually; no spatial partitioning.

**Wrapping** — all movement uses `wrap(v, max)` so the canvas edges are toroidal (objects exit one side, reappear on the other).

**Asteroid splitting** — `Asteroid.split()` returns two new `Asteroid` instances at `size - 1`. Size 1 asteroids return `[]`. Sizes are 1 (small), 2 (medium), 3 (large); constants `RADII`, `SPEEDS`, and `POINTS` are indexed by size.

**Input** — `keys` tracks held keys; `justPressed` tracks single-frame presses (consumed by `pressed(code)` after reading, so firing and restart are edge-triggered, not held).

## Controls

| Key | Action |
|-----|--------|
| `←` `→` | Rotate ship |
| `↑` | Thrust |
| `Space` | Shoot / restart |

