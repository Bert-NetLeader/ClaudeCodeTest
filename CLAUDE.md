# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Games

Open any `.html` file directly in a browser — no build step, server, or dependencies required:

```bash
open shooter.html
open tictactoe.html
```

## Repository Structure

Each game is a **single self-contained HTML file** with embedded CSS and JS — no external assets, frameworks, or dependencies. All rendering uses the Canvas 2D API (`fillRect`-based procedural pixel art) or plain DOM manipulation.

## Git & GitHub Workflow

**Commit and push after every meaningful unit of work** — a feature added, a bug fixed, a refactor completed. Never leave a session with uncommitted changes. This ensures work is never lost and any change can be reverted.

```bash
git add <files>
git commit -m "descriptive message"
git push
```

Commit message rules:
- Use the imperative mood: "Add X", "Fix Y", "Refactor Z"
- First line ≤ 72 characters; describe *what and why*, not *how*
- Always append the co-author trailer:
  `Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>`

Remote: `https://github.com/Bert-NetLeader/ClaudeCodeTest` (branch: `main`)

## shooter.html Architecture

The file is organized into 14 labeled sections (comments mark each):

1. **CONSTANTS** — canvas size (800×600), color palette (`PAL`), physics tuning values
2. **LEVEL_CONFIG** — 5 predefined levels + `getLevelConfig(n)` for infinite procedural scaling
3. **INPUT** — `input.keys{}` + `input.mouse{}`, canvas-relative mouse coords via `getBoundingClientRect`
4. **Particle** class
5. **Bullet** class
6. **Enemy** class — standard (red diamond) and elite variant (magenta, 15% spawn rate, ×3 HP, +3 score)
7. **Player** class — WASD/arrow movement, mouse-aim via `atan2`, shoot cooldown, invincibility frames
8. **SPRITE HELPERS** — `spawnBurst()` for particle explosions
9. **COLLISION** — `circlesOverlap(a, b)` used for all hit detection
10. **WAVE CONTROLLER** — `waveController` singleton: tracks spawned count, spawn timer, delegates to `spawnEnemy()`
11. **SCREEN RENDERERS** — `drawBackground()`, `drawHUD()`, `drawLevelBanner()`, `drawMenu()`, `drawGameOver()`; menu/game-over renderers return button rect for click detection
12. **GAME STATE** — mutable `state` object; `startGame()`, `advanceLevel()`, `triggerGameOver()`, `returnToMenu()`; high score via `localStorage`
13. **GAME LOOP** — `updateGame(dt, ts)` / `drawGame(ts)` / `gameLoop(ts)`; `dt` capped at 50ms
14. **BOOTSTRAP** — initial `requestAnimationFrame(gameLoop)`

### State machine
`MENU` → `BANNER` (2s "get ready") → `PLAYING` → `GAME_OVER` → `MENU`

Level advance fires when `score >= levelConfig.scoreToAdvance && enemies.length === 0 && waveController.done`.

## tictactoe.html Architecture

Single-page two-player game. State is three plain variables (`board`, `current`, `gameOver`) plus a `score` object. `render()` rebuilds the board DOM from scratch on each move. Win detection iterates the `WINS` constant (8 combinations).
