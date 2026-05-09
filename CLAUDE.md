# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**הורה במנוסה: מרוץ הקפה** ("Fleeing Parent: The Coffee Race") — a Hebrew-language, RTL browser game. A parent character escapes children by collecting coffee to stay energized. The entire game lives in a single file: `index.html`.

## Running the Game

No build process or dependencies exist. Open `index.html` directly in any modern browser:

```bash
# Quick local server if needed
python3 -m http.server 8000
# then visit http://localhost:8000
```

There are no tests, no linter, no package manager, and no CI configuration.

## Architecture

Everything is in `index.html` in three sections:

1. **HTML structure** (lines 1–156): Game container with a `<canvas>` for rendering and an `#overlay` div for start/game-over screens. The `#ui-layer` shows live score and energy bar using HTML elements (not canvas).

2. **CSS** (lines 8–119): CSS custom properties (`--bg-color`, `--accent-color`, `--text-color`) drive the theme. The `.hidden` utility class toggles screens. `touch-action: none` and `user-select: none` on `body` are required for mobile gameplay.

3. **JavaScript** (lines 157–454): All game logic in a single inline `<script>`.

### Game Loop & State

- `gameLoop(timestamp)` drives everything via `requestAnimationFrame`. Delta time (`dt`) is capped at 16ms to prevent jumps when the tab is backgrounded.
- Global state: `gameActive`, `score`, `energy`, arrays `children`, `coffees`, `particles`, and `lastTime`/`spawnTimer`/`coffeeTimer`.
- `startGame()` is attached to `window` so HTML `onclick` attributes can call it.

### Entity Classes

- **`Child`**: Spawns off-screen on a random edge, chases the player each frame. Speed scales with `score` (`1.0 + random*1.5 + score/50`). Drains `energy -= 0.8` on collision.
- **`Coffee`**: Spawns randomly on-screen, pulses via `Math.sin`, disappears after 7000ms. Collecting restores `energy += 30` and triggers particles.

### Input Handling

Two independent input modes run simultaneously — keyboard state is tracked in a `keys` object, and mouse/touch set `player.targetX/targetY`. When no keyboard key is active, the player lerps toward the mouse/touch target (`player.speed = 0.15`). Keyboard movement syncs `targetX/Y` to current position to prevent snap-back.

### Difficulty Scaling

Child spawn rate: `Math.max(300, 2000 - score * 30)` ms — accelerates until capped at one child per 300ms. Energy drains at 4 units/second passively, plus collision damage.

## Key Conventions

- The game is Hebrew/RTL — all user-visible strings must be in Hebrew and the `<html>` element must keep `lang="he" dir="rtl"`.
- Emoji are used for all sprites (player `🏃`, children `👶👧👦🧸`, coffee `☕`). Canvas draws them with `ctx.fillText` at `textAlign: center`, `textBaseline: middle`.
- `ctx.globalAlpha` is always reset to `1` after particle rendering — don't break this invariant.
- Max 3 simultaneous coffee items (`coffees.length < 3` guard).
- Energy is clamped: `Math.min(100, energy + 30)` on pickup, `Math.max(0, energy)` in the UI bar.
