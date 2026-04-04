# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Games

Open any `.html` file directly in a browser — no build step, server, or dependencies required:

```bash
open shooter.html
open tictactoe.html
open slotmachine.html
open pool.html
```

## Repository Structure

Each game is a **single self-contained HTML file** with embedded CSS and JS — no external assets, frameworks, or dependencies. All rendering uses the Canvas 2D API (`fillRect`-based procedural pixel art) or plain DOM manipulation. All audio uses the Web Audio API (oscillator/noise-based, no external files).

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

## pool.html Architecture

Canvas-based two-player 8-ball pool. The JS is organized into 15 labeled sections:

1. **CONSTANTS** — canvas (900×560), table geometry (`TX/TY/TW/TH`, `RAIL=36`, `PX/PY/PW/PH`), physics values (`R=12`, `FRICTION=0.987`, `POCKET_R=22`, `MAX_POWER=20`), `POCKETS[6]`, `COLORS[16]`.
2. **CANVAS SETUP** — single `<canvas id="c">`, 2D context.
3. **Ball class** — `num`, `x/y`, `vx/vy`, `pocketed`, `type` (`cue`/`solid`/`eight`/`stripe`). Methods: `applyFriction()`, `wallBounce()` (fires `playWallBounce` on hit), `draw()` (shadow, stripe-clip, number circle, specular highlight).
4. **PHYSICS** — `ballBallCollide(a, b)`: separates overlapping balls, applies elastic equal-mass impulse along collision normal, fires `playBallHit(|rv|)`.
5. **GAME STATE** — `balls[]`, `cueBall`, `currentPlayer`, `playerType[2]`, `gameOver`, `winner`, `inBallInHand`, `breakShot`, `pocketedThisTurn[]`, `msg`/`msgTimer`. `buildRack()` places the standard 5-row triangle (`RACK_ROWS`). `initGame()` resets all state.
6. **POCKET DETECTION & RESOLVE** — `checkPockets()` fires `playPocket()` and appends to `pocketedThisTurn`. `resolveShot()` handles: 8-ball win/loss, scratch (ball-in-hand + `playScratch()`), solid/stripe type assignment on first post-break pot, and turn switching. `endGame(p, text)` fires `playWin()`.
7. **AUDIO** — lazy `AudioContext` via `getAC()`; throttle helper `throttled(id, ms)` prevents sound spam. Six named sounds, all oscillator/noise-based: `playCueStrike(power)`, `playBallHit(relVel)`, `playWallBounce(spd)`, `playPocket()`, `playScratch()`, `playWin()`. `muted` flag toggled by M key.
8. **INPUT** — `mousemove` updates `mouseX/Y` and `shootPower` while dragging; `mousedown` locks `aimAngle` or places ball-in-hand; `mouseup` fires `playCueStrike` + sets cue ball velocity. R restarts, M toggles mute.
9. **DRAW: TABLE** — `roundRect()` helper; wood frame with grain lines, cushion rubber, felt radial gradient, three spots, dashed head string, pocket holes.
10. **DRAW: CUE STICK** — dotted aim line; cue drawn at `angle + PI` (tip near ball, butt extending back); pullback offset = `shootPower * 2.8` capped at 65px; wrap ring and chalk tip.
11. **DRAW: POWER BAR** — vertical bar right of table, green→yellow→red gradient fill proportional to `shootPower / MAX_POWER`.
12. **DRAW: BALL-IN-HAND CURSOR** — semi-transparent white ball follows mouse when placing.
13. **DRAW: HUD** — dark top/bottom panels; player name + type assignment + remaining count; mini ball tray (15 × 8px circles, striped where appropriate, dimmed when pocketed); status message with fade-out; mute indicator.
14. **GAME LOOP** — `update()`: detects moving→stopped transition to call `resolveShot()`; per-frame: move + friction → `checkPockets()` → `wallBounce()` → `ballBallCollide()` (pocket check before wall bounce so corner balls sink cleanly). `draw()` composes all layers. `loop()` drives `requestAnimationFrame`.
15. **BOOTSTRAP** — `initGame()` + `loop()`.

### Physics update order (per frame, while moving)
1. Move all balls (`pos += vel`) + apply friction
2. `checkPockets()` — pocket detection runs first so corner-bound balls sink before wall bounce fires
3. `wallBounce()` per ball
4. `ballBallCollide()` all pairs

### State machine
`IDLE` → shooting (cue ball moving) → all balls stop → `resolveShot()` → `IDLE` or `BALL_IN_HAND` or `GAME_OVER`

Type assignment: first object ball potted after the break determines solid/stripe for both players. On the break itself, no assignment is made regardless of what's potted.
