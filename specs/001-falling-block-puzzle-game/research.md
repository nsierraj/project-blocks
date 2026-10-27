# Research: Falling-Block Puzzle Game (v1)

Phase 0 output for [plan.md](plan.md). It resolves the open technology items from the BRD (O1 to O4) and the details the spec left to planning. No unresolved clarification items remain. Decisions here are the source for test fixtures (Constitution III). This research was done from established guideline rules and platform knowledge; no external lookups were run, so items marked **verify** need a quick check against the cited public reference while writing tests or on a real device.

## 1. Language, build tooling and dependencies

- **Decision**: Strict TypeScript, built and served with Vite. No UI framework and no runtime dependencies. Dev dependencies: Vite, `vite-plugin-pwa`, Vitest, Playwright (with `@axe-core/playwright` for automated accessibility checks), ESLint, TypeScript. Versions: use the current stable release when scaffolding and pin them in the lockfile.
- **Rationale**: The payload budget is 500 KB and first load is under 2 s (NFR-007); a framework would spend a large part of that for a game with seven screens. TypeScript's strict mode and a separate no-DOM config enforce Constitution II mechanically. Vite gives a fast dev loop, a small production build and a standard path to GitHub Pages.
- **Alternatives considered**:
  - React or Svelte for menus: faster to build forms, but adds runtime weight and a second mental model; the screens are simple DOM.
  - Plain JavaScript: smaller toolchain, but loses the compile-time DOM isolation of the core.
  - Phaser or another game engine: heavy, hides the loop and rendering this project exists to show (OB4), and is bigger than the payload budget allows.
  - Rust/WASM core: attractive for determinism, but a larger toolchain and download, and unnecessary at this scale.

## 2. Rendering and UI split

- **Decision**: Canvas 2D for the playfield, ghost, hold and next queue. DOM for menus, HUD text, settings and the gesture overlay. Callouts (T-spin, Quad, back-to-back, combo) are DOM text in an `aria-live="polite"` region plus canvas effects that are skipped under reduce motion.
- **Rationale**: Menus need real focus order, labels and contrast tooling (FR-045, FR-046), which DOM provides for free. The playfield is a 10 by 20 grid, so Canvas 2D is far below the 60 fps limit even on mid-range phones. Canvas is resized by device pixel ratio.
- **Alternatives considered**: DOM or SVG for the board (more nodes to update each frame, harder to keep smooth); WebGL (more code and risk of context-loss handling, no benefit at this size); canvas for menus too (breaks keyboard and screen-reader support).

## 3. Game loop and determinism

- **Decision**:
  - The core advances in **fixed 1 ms ticks** of integer simulation time. The app loop runs on `requestAnimationFrame`, accumulates real elapsed time, and calls the core to advance by whole milliseconds, with a catch-up cap of 100 ms per frame (the game is paused on blur anyway).
  - Inputs reach the core as timestamped events; the core applies each at its timestamp inside the next frame's batch, so input takes effect on the next frame (NFR-006) and Sprint times are exact to the millisecond.
  - The core uses a seeded integer generator (mulberry32 with `Math.imul`), a Fisher-Yates shuffle for each bag, precomputed gravity constants (no `Math.pow` in the simulation, whose results can differ across engines), and no `Date`, `performance` or `Math.random`.
  - The app creates the seed with `crypto.getRandomValues` and passes it to the core; the seed and input log can reproduce a game.
- **Rationale**: SC-006 and SC-010 require identical speed across refresh rates and repeatable times. A 1 ms tick makes "timer to the millisecond" true instead of quantized to 16.7 ms. The per-tick work is tiny (a few counters), so about 1000 ticks per second is cheap.
- **Alternatives considered**: 60 Hz fixed tick (simpler, but Sprint times would only move in 16.7 ms steps); variable timestep (breaks determinism and refresh-rate independence).
- **Pause, countdown and focus**: The core has no pause state. The app stops calling the core when paused, backgrounded, or counting down; game time therefore never advances during pause or the 3 s countdown (FR-032, FR-044). Pausing on `visibilitychange`, `blur` and `pagehide` covers tab switches and backgrounding.

## 4. Rule decisions (test fixtures for Constitution III)

All values are for the core. Items the spec deferred to planning are marked **(decided here)**.

### 4.1 Playfield, spawn and queue

- Grid is 10 columns by 40 rows (20 visible plus 20 hidden above). Spawn is in the hidden rows directly above the visible area, horizontally centered (the I and O pieces use their standard guideline columns), then the piece moves down one cell immediately if free.
- 7-bag: each bag is the seven pieces in a seeded shuffle. The queue always holds at least 5 previewed pieces plus the active one.
- Spawn collision ends the game (FR-005) except in Zen. Lock-out (locking entirely above the visible area) needs no separate rule because the next spawn will collide. **(decided here)**
- No spawn delay and no line-clear delay in simulation time: the next piece spawns and accepts input on the next tick after a lock. Line-clear animation is visual only and never blocks input. **(decided here)**

### 4.2 Gravity (computed values)

Guideline curve: seconds per row = (0.8 - (level - 1) x 0.007) raised to the power (level - 1). The values below were computed and are stored as constants.

| Level | ms per row | Level | ms per row |
|---|---|---|---|
| 1 | 1000.0 | 9 | 93.9 |
| 2 | 793.0 | 10 | 64.2 |
| 3 | 617.8 | 11 | 43.0 |
| 4 | 472.7 | 12 | 28.2 |
| 5 | 355.2 | 13 | 18.2 |
| 6 | 262.0 | 14 | 11.4 |
| 7 | 189.7 | 15 | 7.1 |
| 8 | 134.7 | | |

- **Past level 15 (decided here)**: the level keeps counting up (it still multiplies scores) but gravity stays at the level-15 value. The core may move a piece several rows in one tick at high speed.
- **Level rule**: level = starting level + floor(lines cleared / 10) in Marathon and Ultra (Ultra starts at 1). Sprint stays at level 1. In Zen the level number rises the same way and only affects the score multiplier, since Zen has no gravity. **(decided here)**
- **Sprint** uses level 1 gravity (1000 ms per row) throughout, per the clarification.
- **Soft drop (decided here)**: soft drop multiplies current gravity by 20. A held soft drop scores 1 point per cell moved by soft drop. Cells moved by natural gravity score nothing. Soft drop does not lock instantly.
- **Hard drop**: moves the piece straight to the ghost position and locks it in the same tick (no lock delay). Scores 2 points per cell dropped.
- **Zen**: no gravity at all; the piece moves down only by soft drop or hard drop. Lock delay does not apply in Zen, so a piece resting on the stack stays until the player hard drops. **(decided here)**

### 4.3 Lock delay

- Lock delay is 500 ms. It starts the first tick the active piece cannot move down.
- Any successful move or rotation while the piece is grounded resets the timer to 500 ms and uses one of 15 resets for that piece. The counter **never refreshes** during the life of one piece. **(decided here; this follows the spec text "up to 15 resets per piece" literally. Some guideline implementations refresh the counter when the piece reaches a new lowest row. Switching later is a small change in `game.ts` and its tests.)**
- When the 15 resets are used, the timer stops resetting: the piece locks when the current delay ends. If the piece is moved off a ledge and touches down again after the counter is exhausted, it locks immediately on touchdown. **(decided here)**
- A move that leaves the piece no longer grounded cancels the timer but does not refund resets.
- A failed rotation or move (blocked) does not reset the timer and does not use a reset.

### 4.4 Rotation (SRS)

Two kick tables are used, with the first test always being the no-offset rotation. Offsets are (x, y) with y positive upward. The O piece has no kicks. A rotation succeeds with the first test position that does not collide; if all fail, the piece stays. Counterclockwise uses the reverse transition.

**J, L, S, T, Z**

| Transition | Test 1 | Test 2 | Test 3 | Test 4 | Test 5 |
|---|---|---|---|---|---|
| 0 to R | (0,0) | (-1,0) | (-1,+1) | (0,-2) | (-1,-2) |
| R to 0 | (0,0) | (+1,0) | (+1,-1) | (0,+2) | (+1,+2) |
| R to 2 | (0,0) | (+1,0) | (+1,-1) | (0,+2) | (+1,+2) |
| 2 to R | (0,0) | (-1,0) | (-1,+1) | (0,-2) | (-1,-2) |
| 2 to L | (0,0) | (+1,0) | (+1,+1) | (0,-2) | (+1,-2) |
| L to 2 | (0,0) | (-1,0) | (-1,-1) | (0,+2) | (-1,+2) |
| L to 0 | (0,0) | (-1,0) | (-1,-1) | (0,+2) | (-1,+2) |
| 0 to L | (0,0) | (+1,0) | (+1,+1) | (0,-2) | (+1,-2) |

**I**

| Transition | Test 1 | Test 2 | Test 3 | Test 4 | Test 5 |
|---|---|---|---|---|---|
| 0 to R | (0,0) | (-2,0) | (+1,0) | (-2,-1) | (+1,+2) |
| R to 0 | (0,0) | (+2,0) | (-1,0) | (+2,+1) | (-1,-2) |
| R to 2 | (0,0) | (-1,0) | (+2,0) | (-1,+2) | (+2,-1) |
| 2 to R | (0,0) | (+1,0) | (-2,0) | (+1,-2) | (-2,+1) |
| 2 to L | (0,0) | (+2,0) | (-1,0) | (+2,+1) | (-1,-2) |
| L to 2 | (0,0) | (-2,0) | (+1,0) | (-2,-1) | (+1,+2) |
| L to 0 | (0,0) | (+1,0) | (-2,0) | (+1,-2) | (-2,+1) |
| 0 to L | (0,0) | (-1,0) | (+2,0) | (-1,+2) | (+2,-1) |

**Verify** the tables against the public SRS reference when writing `tests/unit/srs.test.ts`. A transposed sign is the most common bug, so the tests must include a wall kick, a floor kick and a T-spin-triple kick case.

### 4.5 T-spin detection

- Only a T piece can T-spin, and the **last successful action before locking must be a rotation** (a later move or a soft-drop step cancels it). **(decided here)**
- 3-corner rule: look at the four cells diagonal to the T piece's center. Cells outside the board (walls and floor) count as occupied. At least 3 occupied corners is a T-spin.
- **Full vs mini**: it is a full T-spin if both "front" corners (the two on the side the T points toward) are occupied. Otherwise it is a mini. **Exception**: if the final rotation used kick test 5 (the (+/-1, +/-2) offset), it counts as a full T-spin even if the front corners are not both occupied. **(decided here, per guideline)**
- Scoring labels used: T-spin mini (0 lines), T-spin (0 lines), mini single, mini double, T-spin single, double, triple.
- The spec lists no mini double score. **(decided here)**: mini double scores 400 x level, per the guideline.

### 4.6 Scoring

Points use the level **at the moment of the clear**, before the clear's lines are added to the level. **(decided here)**

| Event | Base points |
|---|---|
| Single / Double / Triple / Quad | 100 / 300 / 500 / 800 |
| T-spin mini (0 lines) / T-spin (0 lines) | 100 / 400 |
| T-spin mini single / mini double | 200 / 400 |
| T-spin single / double / triple | 800 / 1200 / 1600 |
| Soft drop | 1 per cell (not multiplied by level) |
| Hard drop | 2 per cell (not multiplied by level) |

- Line-clear and T-spin base points are multiplied by the level.
- **Combo (decided here)**: the combo counter starts at -1. Each lock that clears at least one line adds 1; each lock that clears none resets it to -1. Bonus = 50 x max(combo, 0) x level, so the first clear in a chain earns no bonus and the second earns 50 x 1 x level.
- **Back-to-back (decided here)**: a "difficult" clear is a Quad, or any T-spin or mini T-spin that clears at least one line. The state becomes active after the first difficult clear. A difficult clear while active multiplies **only that clear's line-clear points** by 1.5 (not combo or drop points). A non-difficult line clear (Single, Double or Triple without a T-spin) deactivates it. A lock that clears no lines (including a T-spin with 0 lines) does not change it.
- Perfect clear (F19) and 180-degree rotation (F12) are Could items and are **deferred**: the scoring module has a single extension point for a perfect-clear bonus but no points are defined in v1.

**Worked examples (become unit-test fixtures).** Each row assumes the state shown and that drop points are listed separately.

| # | Level | State before | Event | Line-clear points | Combo bonus | Total for this lock |
|---|---|---|---|---|---|---|
| 1 | 1 | none | Single | 100 | 0 | 100 |
| 2 | 1 | none | Quad | 800 | 0 | 800 |
| 3 | 1 | back-to-back active | Quad | 800 x 1.5 = 1200 | 0 | 1200 |
| 4 | 1 | back-to-back active | T-spin double | 1200 x 1.5 = 1800 | 0 | 1800 |
| 5 | 1 | combo 0 (one clear before) | Single | 100 | 50 x 1 x 1 = 50 | 150 |
| 6 | 1 | combo 1 | Single | 100 | 50 x 2 x 1 = 100 | 200 |
| 7 | 3 | none | Double | 900 | 0 | 900 |
| 8 | 3 | none | Quad | 2400 | 0 | 2400 |
| 9 | 3 | back-to-back active | Quad | 2400 x 1.5 = 3600 | 0 | 3600 |
| 10 | 3 | none | T-spin double | 3600 | 0 | 3600 |
| 11 | 3 | back-to-back active | T-spin double | 3600 x 1.5 = 5400 | 0 | 5400 |
| 12 | 3 | none | T-spin mini single | 600 | 0 | 600 |
| 13 | 3 | combo 1 | Double | 900 | 50 x 2 x 3 = 300 | 1200 |
| 14 | 3 | combo 0, back-to-back active | Quad | 3600 | 50 x 1 x 3 = 150 | 3750 |
| 15 | 3 | any | T-spin, 0 lines | 400 x 3 = 1200 | 0 (combo resets) | 1200 |
| 16 | any | any | Hard drop 10 cells | n/a | n/a | 20 |
| 17 | any | any | Soft drop 6 cells | n/a | n/a | 6 |

In rows 5 and 6, "combo N" is the counter value before this lock; the lock's own increment gives the value used in the bonus.

### 4.7 Hold, queue and top-out

- Hold is allowed once per dropped piece and re-enabled when a piece locks. Using hold with an empty slot takes the next piece from the queue. The swapped-in piece appears at the spawn position in its spawn orientation; if that collides, the game ends.
- Top-out in Sprint is shown on Results as "did not finish" and is **not ranked or saved**. In Marathon and Ultra a top-out ends the game and the score is ranked normally. In Zen a top-out clears the board instead. **(decided here)**

- **Quit (decided here)**: a quit from Pause in Marathon, Sprint or Ultra asks for confirmation and then discards the game: no saved score and no Results screen. Only a Zen quit shows Results and is saved.

### 4.8 Modes (summary)

| Mode | Start level | Ends when | Gravity | Ranked by |
|---|---|---|---|---|
| Marathon | chosen, 1 to 15 | 150 lines (unless endless) or top-out | Table, plateau at 15 | Score; separate lists for 150-line and endless |
| Sprint | 1 | 40 lines (top-out is a non-ranked fail) | Level 1, fixed | Time, fastest first |
| Ultra | 1 | 180 000 ms of game time, or top-out | Table, level rises every 10 lines | Score |
| Zen | 1 | Player quits from Pause | None | Lines cleared |

Ties keep both entries; the earlier date ranks first. The Sprint time is the game-clock time at the lock that clears the 40th line.

### 4.9 Keyboard repeat defaults

- DAS default 167 ms, ARR default 33 ms. Setting ranges: DAS 50 to 300 ms, ARR 0 to 100 ms (0 moves the piece to the wall instantly while held). **(decided here; guideline-like starting values.)**
- If both horizontal keys are held, the most recently pressed direction wins; releasing it resumes the other if still held.
- DAS and ARR live in the core, driven by press and release events, so replays reproduce them.

## 5. Input

- **Keyboard**: `keydown`/`keyup` on the window, using `KeyboardEvent.code` for layout-independent defaults, with remapping stored in settings and a conflict rule (assigning a used key swaps the two actions). Auto-repeat events from the OS are ignored; the core handles repeat.
- **Gestures**: Pointer Events on the game surface with CSS `touch-action: none` and `overscroll-behavior: none`, plus non-passive `touchmove` `preventDefault` on that surface only, so menus keep normal scrolling and focus behavior (FR-029). Gestures are recognised only on the game surface; both input types are always active (clarification 4).
- **Gesture definitions (starting values, tuned on real devices, R2)** at sensitivity 1.0:
  - Tap: up within 200 ms and moved less than 10 px; right half rotates clockwise, left half rotates counterclockwise.
  - Drag horizontally: the piece follows the finger one column per cell-width of travel (emitting one-shot left/right steps).
  - Slow drag down: after a 12 px dead zone, soft drop is held while the finger keeps moving down below the flick speed.
  - Flick down: at least 40 px at 0.8 px/ms or faster hard drops.
  - Swipe up: at least 48 px, mostly upward, at least 0.3 px/ms, uses hold.
  - Sensitivity setting scales distances and speeds between 0.5 and 1.5.
  - Only the primary pointer is tracked; extra touches are ignored.
- **iOS Safari tab limits**: the system edge swipe-back may not be preventable from a page; the installed standalone app avoids it. Treated as a known limitation to confirm on a real iPhone (plan, Complexity Tracking).

## 6. Audio and haptics (resolves BRD O4)

- **Decision**: Synthesize all sound effects at runtime with Web Audio (short oscillator and noise envelopes), one function per effect: move, rotate, hold, lock, line clear (1 to 4 lines), T-spin, level up, game over. No audio files.
- **Rationale**: Zero licence risk (Constitution I), nearly zero download weight (NFR-007), and fully original. The audio context is created and resumed on the first user interaction, so sound starts silent until then (FR-041). `ASSETS.md` records that all sound is synthesized in `src/audio/sfx.ts`.
- **Alternatives considered**: royalty-free sample pack (needs licence tracking and adds payload); creating recordings (time cost, needs an editor).
- **Haptics**: `navigator.vibrate` behind feature detection; short pulse on line clears and hard drops; skipped silently when unavailable, including iOS Safari (FR-040).

## 7. Storage

- **Decision**: A thin wrapper over `localStorage` with a key prefix (`blocks:`), a stored `schemaVersion`, JSON validation on read, and an in-memory fallback when access throws (private mode, blocked storage, full quota). Schema in [contracts/storage-schema.md](contracts/storage-schema.md).
- **Why prefix and version**: GitHub Pages serves all of a user's project sites from one origin, so unprefixed keys can collide with another project (and another project could read these). Versioning plus a migration function satisfies NFR on updates not breaking saved data (FR-049). Invalid or unknown data is replaced with defaults rather than crashing.
- **Alternatives considered**: IndexedDB (async and heavier for a few kilobytes); cookies (forbidden by FR-047).

## 8. Offline, install and updates (PWA)

- **Decision**: `vite-plugin-pwa` in generate-service-worker mode with precaching of the whole build. A new service worker **waits** until all tabs of the old version close, so an update activates on next launch and never reloads a game in progress (FR-049). `cacheId` is set to the app id so caches are namespaced; the service worker deletes only caches it created.
- **Scope**: The service worker, manifest `start_url` and `scope` all use the deployment base path (section 9). Icons are original and generated, recorded in `ASSETS.md`.
- **Alternatives considered**: hand-written service worker (more code and a higher chance of update bugs); update-on-prompt UI (extra screen the spec doesn't ask for).
- **Verify** on a real iPhone: install from Safari "Add to Home Screen", then airplane mode, then a full game (SC-003). iOS can evict storage for sites not used recently (R5); scores fall back gracefully.

## 9. Hosting on GitHub Pages

- **Decision**: Deploy with a GitHub Actions workflow that builds with a `BASE_PATH` environment variable (`/<repo-name>/`) and publishes the `dist` folder to Pages. The base path feeds Vite's `base`, the manifest, and the service worker scope. All internal URLs are relative to the base.
- **Consequences**: The repository name becomes the public URL path, so **the repository must be renamed to the final game name before the first deploy** (BRD O1, plan Complexity Tracking). Deep links are not needed (screens are an in-page router with no history entries beyond what the app needs), so no `404.html` fallback is required.
- **Alternatives considered**: custom domain (not needed for a portfolio piece); Netlify/Vercel (the BRD names GitHub Pages).

## 10. Visual style (resolves BRD O3)

- **Decision**: Flat, rounded tiles on a plain background (no background image or gradient, to keep contrast checkable). Seven colors from the Okabe-Ito colorblind-safe palette, each paired with a distinct inner pattern so pieces differ in grayscale (FR-002, FR-042):

| Piece | Color | Pattern |
|---|---|---|
| I | sky blue | horizontal stripes |
| O | yellow | dots |
| T | reddish purple | diagonal hatch |
| S | bluish green | zigzag |
| Z | vermillion | cross-hatch |
| J | blue | vertical stripes |
| L | orange | inner ring |

- Light and dark themes are CSS custom properties switched by `prefers-color-scheme` plus a manual override saved in settings (FR-039). Text and UI component contrast is checked against WCAG 2.2 AA with automated and manual checks. The ghost piece is an outline, the held piece is dimmed when hold is used.
- **Fonts**: system font stack only, no web fonts (payload).

## 11. Testing and budgets

- **Automated in CI**: typecheck with both `tsconfig` files (the core one with no DOM lib), ESLint with an import restriction that stops `src/core` importing other source folders, Vitest unit tests (SRS, T-spin, scoring, bag, gravity, modes, storage), replay tests (fixed seed plus timestamped inputs must give identical score, lines and time; a 60, 90 and 120 Hz frame schedule must give the same result), production build, a gzip size check of the whole `dist` against the 500 KB budget (the working goal is far below it), Playwright smoke tests in Chromium, Firefox and WebKit (keyboard play, touch emulation, offline reload), and axe checks on menu screens.
- **End-to-end test hook**: the Playwright game tests run against a build made with `vite build --mode e2e`, which includes `src/app/test-hook.ts` (fast-forward of game time and scripted input). The normal production build does not contain it, so the shipped bundle and the size check are unaffected. The offline test runs on the normal production build because it needs the real service worker.
- **Manual (needs real devices or throttling)**: the frame-rate rule from NFR-004 on the two reference phones, first load under 2 s with the DevTools "Slow 4G" preset and an empty cache (NFR-007; confirm the preset's current bandwidth and latency values), iPhone and Android install plus offline play, gesture feel and edge-swipe behavior on iOS Safari, a screen-reader pass, and contrast spot checks. Playwright's WebKit is not iOS Safari, so it does not replace these.

## 12. Naming and IP (BRD O1)

- The working name "Project Blocks" is used for the manifest, page title and README until a final name is chosen. App id for storage and cache names is `blocks`. The final name must not contain the genre trademark.
- Open item: the repository directory and BRD file names and the BRD's body text mention the trademark. This is recorded in the plan's Complexity Tracking and must be resolved before the first public push.
