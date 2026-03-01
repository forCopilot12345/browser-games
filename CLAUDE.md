# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the games

No build step or server required. Open any HTML file directly in a browser:

```bash
start shooter.html      # Windows
start tictactoe.html
```

Or double-click the file in Explorer. All logic is self-contained in each file.

## Repository structure

Each game is a **single self-contained HTML file** — HTML, CSS, and JavaScript all in one file, no external dependencies, no bundler, no framework.

| File | Description |
|---|---|
| `shooter.html` | Top-down shooter — full game with 5 levels, boss, particle system |
| `tictactoe.html` | 2-player tic-tac-toe with score tracking |

## shooter.html architecture

The game runs on a fixed 800×600 canvas, scaled to fit the window via CSS. Everything lives in module-style sections separated by banner comments:

**Global state (module-level arrays, not encapsulated):**
- `bullets[]` — all active bullets (player + enemy)
- `particles[]` — all active particles
- `bgTiles[]`, `menuParticles[]` — static decoration arrays initialized once at load

**Core systems (plain functions, not classes):**
- `fireBullet / updateBullets / drawBullets` — bullet lifecycle
- `spawnParticles / spawnMuzzleFlash / updateParticles / drawParticles` — particle lifecycle
- `createEnemy / updateEnemies / drawEnemies` — enemy factory + per-frame logic
- `createPlayer()` — returns a player object with `update()` and `draw()` methods
- `draw*` functions (`drawPlayer`, `drawGrunt`, `drawTank`, `drawDasher`, `drawBoss`, `drawMiniHpBar`, `drawHUD`, `drawBackground`) — all rendering, all procedural Canvas 2D API, no image assets

**Game state machine (`game` object):**
```
MENU → PLAYING → LEVEL_COMPLETE → PLAYING (next level)
                               ↘ GAME_OVER → MENU
                 VICTORY (after level 5) → MENU
```
States are driven by `game.update()` / `game.draw()` called every frame from `requestAnimationFrame`.

**Level progression:**
The `LEVELS` array (top of file) defines all 5 levels. `game.loadLevel()` builds a `spawnQueue[]` from `LEVELS[levelIndex].types`, shuffles it, then drains it one enemy at a time on a timer. Level ends when both `spawnQueue` and `enemies` are empty.

**Enemy AI per type:**
- `grunt` / `tank` — steer toward player each frame (seek behavior with velocity damping)
- `dasher` — slow drift, then periodic charge (locks angle at charge start)
- `boss` — orbit + periodic charge + spread bullet fire; fire rate doubles below 50% HP

**Timing:** All animation uses a `frame` counter incremented each tick, driving `Math.sin(frame * k)` oscillations. Game runs at whatever `requestAnimationFrame` provides (~60 fps). No delta-time normalization.

**Mouse-to-canvas coordinate mapping:** Mouse events are scaled from CSS pixels to canvas pixels using `canvas.getBoundingClientRect()` + the CSS scale ratio each frame.

## Git workflow

- Branch: `master`, remote: `origin` (GitHub: `forCopilot12345/browser-games`)
- Commit and push after every meaningful change
- Commit messages: imperative present tense, descriptive subject line, bullet body for multi-part changes
