# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Games

Open any `.html` file directly in a browser — no build step, server, or dependencies required:

```bash
open shooter.html
open tictactoe.html
open slotmachine.html
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

## slotmachine.html Architecture

DOM-based slot machine (no canvas). The JS is organized into labeled sections:

1. **SYMBOLS** — 9 symbols with `id`, `label`, `cls`, `mult3`, `mult2`, `weight`; built into a weighted `POOL` array for random draws. Special symbols: `wild` (🌟, weight 2) substitutes for any symbol except seven; `scatter` (💎, weight 1) is position-independent. Seven's `mult3` is 0 — its payout is overridden by the progressive jackpot at resolve time.
2. **PAYLINES** — 5 entries (`rows[3]` + `name` + `color` + `indicatorId`): TOP, MID, BOT, DIAG↘, DIAG↗. `rows[r]` = which row index to read from reel `r`.
3. **CONSTANTS** — `BET_OPTIONS`, `FREE_SPIN_COUNT` (8), `JACKPOT_SEED` (200).
4. **STATE** — `bet`, `credits`, `jackpot`, `freeSpins`, `freeSpinWinnings`, `spinning`, `muted`, `audioCtx`.
5. **DOM REFS** — `machineEl`, `cellEls[reel][row]`, `reelEls[reel]`, `plIndicators[]`.
6. **AUDIO** — `getCtx()` lazily creates `AudioContext`; `tone(freq, type, gain, duration, startTime)` is the shared primitive. Named functions: `playSpinStart`, `playTick`, `playWin(big)`, `playJackpot`, `playNoWin`, `playScatterChime`, `playFreeSpinFanfare`. All oscillator-based, no external files.
7. **PERSISTENCE** — `loadCredits/saveCredits` and `loadJackpot/saveJackpot` via `localStorage` keys `slotmachine_credits` / `slotmachine_jackpot`.
8. **JACKPOT DISPLAY** — `addToJackpot(amount)` grows pot and triggers CSS bump animation; called each paid spin with `Math.ceil(bet * 0.5)`.
9. **HELPERS** — `setCell(r, c, sym)` rebuilds className + textContent; `flashCell(r, c, color)` sets `--flash-color` CSS var and toggles `win-flash` class; `updateBetUI()` disables unaffordable bet buttons and auto-reduces `bet`.
10. **WIN EVALUATION** — `evaluateLine(results, line)` handles wild substitution and cherry pair; returns `{ payout, isJackpot, cells }` or `null`. `countScatters(results)` / `getScatterCells(results)` scan the full 3×3 grid independently of paylines.
11. **FREE SPINS** — `startFreeSpins(count)` adds `.free-spin-mode` to `.machine` (purple glow); `endFreeSpins()` removes it and reports `freeSpinWinnings`. Retriggerable: 3 scatters during free spins adds `FREE_SPIN_COUNT` more.
12. **SPIN** — pre-determines full `results[3][3]` grid, runs blur+cycling animation, stops reels left→right (800/1400/2000 ms), resolves at 2200 ms. Jackpot payout overrides `result.payout` inline before summing. Scatter payout (`5 * bet` for 2+) added on top of payline wins.
13. **INPUT** — spin button click, bet button clicks, spacebar, mute toggle.

### State machine
`IDLE` → spinning (locked) → resolve → `IDLE` or `FREE_SPIN` loop → `GAME_OVER`

Free spins: `freeSpins > 0` suppresses bet deduction and jackpot contribution. Scatter retrigger during free spins increments `freeSpins` rather than calling `startFreeSpins` again.

### Key CSS patterns
- `.machine.free-spin-mode` — transitions border/shadow to purple glow.
- `.cell.row-{0,1,2}` — per-row tint matches payline colors (blue/red/green).
- `.cell.wild-sym` / `.cell.scatter-sym` — gold/cyan border overrides via `!important`.
- `.cell.win-flash` — uses `--flash-color` CSS custom property set per-cell for per-payline colored flash.
