---

description: "Task list for the Falling-Block Puzzle Game (v1)"
---

# Tasks: Falling-Block Puzzle Game (v1)

**Input**: Design documents from `/specs/001-falling-block-puzzle-game/`

**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [contracts/](contracts/), [quickstart.md](quickstart.md)

**Tests**: Included. Constitution III requires automated tests for SRS, T-spin, scoring and the 7-bag randomizer to be written before or with the code they cover, and the plan relies on replay and smoke tests. Within each story, write the test tasks first and confirm they fail before implementing.

**Organization**: Tasks are grouped by user story. The stories build on one shared core, so they are cumulative in practice; see Dependencies & Execution Order.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependency on an incomplete task)
- **[Story]**: US1 to US5, matching the user stories in the spec
- File paths are relative to the repository root
- Each task ends with the spec IDs it satisfies (FR, NFR or SC; each carries its BRD ID in `spec.md`), or `(infrastructure)` (Constitution: tasks MUST cite requirement IDs)
- Rule values (gravity table, kick tables, scoring examples, lock rules) are in [research.md](research.md) section 4; interface shapes are in [contracts/](contracts/)

## Path Conventions

Single project at the repository root, as laid out in [plan.md](plan.md): `src/core`, `src/input`, `src/render`, `src/audio`, `src/haptics`, `src/storage`, `src/ui`, `src/app`, `src/pwa`, `tests/unit`, `tests/replay`, `tests/e2e`, `scripts`.

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization, tooling and the guards that enforce the constitution

- [ ] T001 Resolve the repository naming conflict (Constitution I, option (a) chosen 2026-10-26). The BRD file is already renamed to `blocks-brd.md` and its body mentions reworded. Remaining owner steps: rename the working directory, whose current name contains the genre trademark (suggested new name `project-blocks`), outside the session and reopen the session there, and create the GitHub repository under the final game name (BRD O1). Then search tracked files, file names and the directory path to confirm the genre trademark appears nowhere. Blocks any public push, T104 and T105 (renumbered below) (FR-050, SC-009)
- [ ] T002 Initialize the npm project in `package.json` with scripts `dev`, `build`, `preview`, `typecheck`, `lint`, `test`, `e2e`, `check-size`, and dev dependencies TypeScript, Vite, `vite-plugin-pwa`, Vitest, `@playwright/test`, `@axe-core/playwright`, ESLint and `typescript-eslint` (current stable versions, locked in the lockfile) (infrastructure)
- [ ] T003 [P] Create `tsconfig.json` (strict, DOM lib, includes `src` and `tests`) and `tsconfig.core.json` (strict, ES library only with no DOM lib, includes only `src/core`) so any DOM reference in the core fails `npm run typecheck` (NFR-009)
- [ ] T004 [P] Create `vite.config.ts` reading the deployment base path from the `BASE_PATH` environment variable (default `/`) setting the production build target, and defining an `e2e` build mode that includes `src/app/test-hook.ts` (the normal production build never includes it) (NFR-008)
- [ ] T005 [P] Create `eslint.config.js` with a restriction that files in `src/core/**` cannot import from `src/input`, `src/render`, `src/audio`, `src/haptics`, `src/storage`, `src/ui`, `src/app` or `src/pwa`, and cannot use `Date`, `performance` or `Math.random` (NFR-009)
- [ ] T006 [P] Create `vitest.config.ts` covering `tests/unit` and `tests/replay` (Node environment; tests stub browser globals themselves) (NFR-009)
- [ ] T007 [P] Create `playwright.config.ts` with Chromium, Firefox and WebKit desktop projects plus a mobile-emulation project with touch, serving the `--mode e2e` build for the game tests and the normal production build for the offline test (NFR-001)
- [ ] T008 [P] Create `scripts/check-size.mjs` that gzips every file in `dist/` and fails if the total exceeds 500 KB (NFR-007)
- [ ] T009 [P] Create `.gitignore` (node_modules, dist, test output), a skeleton `README.md` and a skeleton `ASSETS.md` listing source and licence for every asset (initially: no external assets) (FR-050, FR-051)
- [ ] T010 Create `index.html` with the provisional title "Project Blocks", viewport meta with `viewport-fit=cover`, light and dark `theme-color` metas, an `#app` root and a `noscript` message (FR-050, NFR-002)
- [ ] T011 [P] Create `.github/workflows/ci.yml` running install, `typecheck`, `lint`, `test`, `build`, `check-size` and `e2e` on push and pull request (no deploy job yet; see T105) (infrastructure)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared types, deterministic random numbers, piece data, theme tokens and the screen router that every story uses

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T012 [P] Define shared core types in `src/core/types.ts`: `PieceType`, `Rotation`, `Cell`, `Action`, `InputEvent`, `GameEvent`, `Snapshot`, `Mode`, exactly as in [contracts/core-api.md](contracts/core-api.md) (NFR-009)
- [ ] T013 [P] Write failing tests in `tests/unit/rng.test.ts`: fixed seed gives a fixed known sequence, different seeds differ, state is reproducible after copying (NFR-005, SC-010)
- [ ] T014 [P] Write failing tests in `tests/unit/pieces.test.ts`: each of the 7 types has 4 cells in all 4 rotation states, spawn cells fit in the hidden rows, I and O use their documented spawn columns (FR-002, FR-004)
- [ ] T015 [P] Implement the seeded generator (mulberry32 with `Math.imul`) in `src/core/rng.ts` (FR-003, SC-010)
- [ ] T016 [P] Implement piece shapes, rotation states and spawn data (no color values) in `src/core/pieces.ts` (FR-002, FR-004)
- [ ] T017 [P] Create theme and focus design tokens as CSS custom properties for light and dark in `src/ui/styles/tokens.css`, including a visible focus outline and the `prefers-color-scheme` rules (FR-039, FR-045, FR-046)
- [ ] T018 [P] Define default keyboard bindings (by `KeyboardEvent.code`) from [contracts/input-actions.md](contracts/input-actions.md) in `src/input/actions.ts` (FR-026, FR-027)
- [ ] T019 Create the screen router with focus management (moves focus to the new screen heading on every change) in `src/ui/router.ts` and an application entry skeleton in `src/app/main.ts` (FR-031, FR-045)

**Checkpoint**: Foundation ready - user story work can begin

---

## Phase 3: User Story 1 - Play a Marathon game with the keyboard (Priority: P1) 🎯 MVP

**Goal**: A player on desktop starts Marathon, plays a full game with the keyboard (spawn, move, rotate with SRS kicks, lock delay, line clears, levels, game over or 150-line completion).

**Independent Test**: Start Marathon, play with the default keys until game over, and confirm each behavior in the story's acceptance scenarios; `npm test` passes the core tests and `npm run e2e` passes the Marathon smoke test.

### Tests for User Story 1 (write first, confirm they FAIL) ⚠️

- [ ] T020 [P] [US1] Write failing tests in `tests/unit/bag.test.ts`: every group of 7 from the start of a bag has one of each type across many seeds, same seed gives the same order, the queue always has at least 5 pieces ready (FR-003)
- [ ] T021 [P] [US1] Write failing tests in `tests/unit/srs.test.ts` using the full kick tables from [research.md](research.md) section 4.4: every transition for the I table and the J/L/S/T/Z table, O has no kicks, a wall kick, a floor kick, a failed rotation leaves the piece unchanged, counterclockwise uses the reverse transition (FR-007)
- [ ] T022 [P] [US1] Write failing tests in `tests/unit/board.test.ts`: collision against walls, floor and blocks, locking a piece, clearing one to four full rows with rows above shifting down, hidden rows participate in collision (FR-001, FR-013)
- [ ] T023 [P] [US1] Write failing tests in `tests/unit/gravity.test.ts`: the level 1 to 15 values from [research.md](research.md) section 4.2, level 16 and above use the level 15 value, soft drop is 20 times gravity (FR-021)
- [ ] T024 [P] [US1] Write failing tests in `tests/unit/lock-delay.test.ts`: lock at 500 ms after touchdown, move or rotate resets up to 15 times, the counter never refreshes, after 15 a new touchdown locks immediately, moving off a ledge cancels the timer without refunding resets, a blocked move or rotation does not reset, hard drop locks immediately (FR-008)
- [ ] T025 [P] [US1] Write failing tests in `tests/unit/das-arr.test.ts`: DAS delay then ARR repeat timing, ARR 0 slides to the wall, latest pressed direction wins and releasing it resumes the other, release before DAS moves exactly one column (FR-027)
- [ ] T026 [P] [US1] Write failing tests in `tests/unit/marathon.test.ts`: starting level 1 to 15, level rises every 10 lines, ends at 150 lines unless endless, spawn collision ends the game (FR-005, FR-021)
- [ ] T027 [P] [US1] Write failing replay tests in `tests/replay/determinism.test.ts`: same seed, options and timestamped inputs give identical snapshots and events; splitting `advance` calls to simulate 60, 90 and 120 Hz frame schedules gives identical results (NFR-005, NFR-006, SC-006, SC-010)

### Implementation for User Story 1

- [ ] T028 [P] [US1] Implement the seeded 7-bag and 5-piece preview queue in `src/core/bag.ts` (FR-003, FR-011)
- [ ] T029 [P] [US1] Implement SRS rotation states and both wall-kick tables in `src/core/srs.ts` (FR-007)
- [ ] T030 [P] [US1] Implement the 10 by 40 playfield, collision, locking and line clearing in `src/core/board.ts` (FR-001, FR-013)
- [ ] T031 [P] [US1] Implement the per-level gravity constants and soft-drop factor in `src/core/gravity.ts` (FR-021, FR-022)
- [ ] T032 [US1] Implement `createGame`, `advance`, `snapshot` and `drainEvents` in `src/core/game.ts`: 1 ms ticks, spawning, gravity, soft and hard drop, SRS movement and rotation, lock delay with the 15-reset rule, line clears, top-out (depends on T028 to T031) (FR-004, FR-005, FR-006, FR-007, FR-008, FR-013, FR-018, NFR-005, NFR-006, NFR-009)
- [ ] T033 [US1] Add held-action handling and DAS/ARR to `src/core/game.ts` per [research.md](research.md) section 4.9 (same file as T032) (FR-006, FR-027)
- [ ] T034 [US1] Implement Marathon rules (start level, level every 10 lines, 150-line end, endless variant) in `src/core/modes.ts` and apply them in `src/core/game.ts` (FR-021)
- [ ] T035 [P] [US1] Implement the keyboard adapter in `src/input/keyboard.ts`: key events become timestamped `InputEvent`s, OS key repeat is ignored, held keys release on blur (FR-026, FR-027, NFR-006)
- [ ] T036 [P] [US1] Implement the Okabe-Ito colors and the seven distinct tile patterns (table in [research.md](research.md) section 10) for light and dark in `src/render/palette.ts` (FR-002, FR-042)
- [ ] T037 [P] [US1] Implement the Canvas 2D renderer for the playfield and active piece, scaled by device pixel ratio, in `src/render/canvas.ts` (FR-001, FR-002, NFR-004)
- [ ] T038 [US1] Implement the animation loop in `src/app/loop.ts`: accumulates real time into whole-millisecond `advance` calls with a 100 ms catch-up cap, batches input per frame, takes the seed from `crypto.getRandomValues` with a `?seed=` override for tests (NFR-004, NFR-005, NFR-006)
- [ ] T039 [US1] Create the Home screen with a Marathon start button, starting level 1 to 15 and the endless option in `src/ui/screens/home.ts` (FR-021, FR-031)
- [ ] T040 [US1] Create the Game screen with the canvas, a HUD for score, level and lines, and an end-of-game panel with "Play again" and "Home" in `src/ui/screens/game.ts` (FR-031)
- [ ] T041 [US1] Wire Home, Game, loop, keyboard and renderer together in `src/app/main.ts` (FR-031)
- [ ] T042 [US1] Create the e2e-only test hook in `src/app/test-hook.ts`, included only in `vite build --mode e2e`: fast-forward game time by a number of milliseconds and feed scripted timestamped inputs to the running game, so end-to-end tests can finish Sprint (40 lines) and Ultra (180 s) quickly and script exact scoring scenarios; the normal production build and the size check must not include it (infrastructure)
- [ ] T043 [P] [US1] Write the Marathon smoke test in `tests/e2e/marathon-keyboard.spec.ts`: start a seeded game, move, rotate, hard drop, observe score and lines change and a game over after stacking to the top, using the e2e test hook (T042) (FR-005, SC-001)

**Checkpoint**: User Story 1 is a complete, playable keyboard Marathon game. This is the MVP.

---

## Phase 4: User Story 2 - Full mechanics: hold, ghost, next queue and bonus scoring (Priority: P2)

**Goal**: Ghost piece, hold, 5-piece next queue, and scoring with T-spins, combos, back-to-back and drop points, with callouts.

**Independent Test**: Play Marathon and perform each scoring event; points and callouts match [research.md](research.md) section 4.6; `tests/unit/scoring.test.ts` and `tests/unit/tspin.test.ts` pass.

### Tests for User Story 2 (write first, confirm they FAIL) ⚠️

- [ ] T044 [P] [US2] Write failing tests in `tests/unit/tspin.test.ts`: 3-corner rule with walls and floor counting as occupied, full versus mini by front corners, the kick-test-5 exception forces a full T-spin, a move or soft drop after the rotation cancels it, non-T pieces never T-spin (FR-015)
- [ ] T045 [P] [US2] Write failing tests in `tests/unit/scoring.test.ts`: all 17 worked examples from [research.md](research.md) section 4.6, mini double is 400 times level, difficult clear classification, back-to-back multiplies only line-clear points, a no-line lock keeps back-to-back but resets combo, points use the level at the time of the clear, soft drop 1 and hard drop 2 per cell (FR-014, FR-015, FR-016, FR-017, FR-018)
- [ ] T046 [P] [US2] Write failing tests in `tests/unit/hold.test.ts`: hold once per piece, re-enabled after lock, empty hold takes the next piece, swapped piece spawns in spawn orientation, a blocked spawn ends the game (FR-010)
- [ ] T047 [P] [US2] Write failing tests in `tests/unit/queue-ghost.test.ts`: ghost cells equal the hard-drop landing position, the next list always has 5 entries in stable order across bag refills (FR-009, FR-011)

### Implementation for User Story 2

- [ ] T048 [P] [US2] Implement T-spin detection (`detectTSpin`) in `src/core/tspin.ts` (FR-015)
- [ ] T049 [P] [US2] Implement scoring (`scoreClear`, combo, back-to-back, drop points) in `src/core/scoring.ts` with a single perfect-clear extension point that awards nothing in v1 (FR-014, FR-015, FR-016, FR-017, FR-018, FR-019)
- [ ] T050 [US2] Integrate hold, ghost, 5-piece next list, scoring, combo, back-to-back and the `lineClear`, `tspin` and `levelUp` events into `src/core/game.ts` (depends on T048, T049) (FR-009, FR-010, FR-011, FR-014, FR-015, FR-016, FR-017, FR-018)
- [ ] T051 [US2] Extend the renderer to draw the ghost outline, the hold panel (dimmed when used) and the 5-piece next queue in `src/render/canvas.ts` (FR-009, FR-010, FR-011)
- [ ] T052 [US2] Implement callouts for T-spins, Quads, back-to-back and combos in `src/render/effects.ts` and an `aria-live="polite"` announcer, driven by game events through `src/app/feedback.ts` (FR-020)
- [ ] T053 [P] [US2] Write the mechanics smoke test in `tests/e2e/mechanics.spec.ts`: seeded game shows the ghost and 5 queue entries, hold swaps once, a scripted line clear (scripted input from the e2e test hook, T042) updates the score by the expected amount (FR-009, FR-010, FR-011, FR-014)

**Checkpoint**: User Stories 1 and 2 together give the full guideline-style Marathon game.

---

## Phase 5: User Story 3 - All four modes with results, high scores and settings (Priority: P3)

**Goal**: Marathon, Sprint, Ultra and Zen; Pause, Results, High Scores, Settings and How to Play screens; scores and settings saved locally and safe when storage is blocked.

**Independent Test**: Play one short game in each mode, check Results, reload to confirm saved data, and block storage to confirm the game still runs; mode, storage, score and settings tests pass.

### Tests for User Story 3 (write first, confirm they FAIL) ⚠️

- [ ] T054 [P] [US3] Write failing tests in `tests/unit/modes.test.ts`: Sprint ends at 40 lines with level 1 gravity and an unranked fail on top-out, Ultra ends at 180000 ms of game time starting at level 1 and levelling every 10 lines, Zen has no gravity and pieces lock only on hard drop, clears the board on top-out instead of ending, and ends only on `quit()` (FR-022, FR-023, FR-024)
- [ ] T055 [P] [US3] Write failing tests in `tests/unit/store.test.ts`: keys are prefixed `blocks:`, a throwing storage falls back to memory, corrupt JSON in one key leaves the others, older schema versions migrate, a newer version is left untouched, reset removes only `blocks:` keys (FR-035, FR-037, FR-049)
- [ ] T056 [P] [US3] Write failing tests in `tests/unit/scores.test.ts`: top 10 per list, ranking keys (score, time ascending, lines), equal values kept with the earlier date first, separate `marathon-150` and `marathon-endless` lists, Sprint stores only completed games, personal-best detection (FR-034)
- [ ] T057 [P] [US3] Write failing tests in `tests/unit/settings.test.ts`: each field's range validation falls back to its default, assigning a used key swaps the two actions, defaults match [contracts/storage-schema.md](contracts/storage-schema.md) (FR-027, FR-035)

### Implementation for User Story 3

- [ ] T058 [US3] Add the Sprint, Ultra and Zen mode configurations and end rules to `src/core/modes.ts` (FR-022, FR-023, FR-024)
- [ ] T059 [US3] Implement Zen (no gravity, no lock delay, board clear on top-out), `quit()` and the Sprint unranked-fail result in `src/core/game.ts` (depends on T058) (FR-024, FR-025)
- [ ] T060 [P] [US3] Implement the prefixed, versioned `localStorage` wrapper with in-memory fallback in `src/storage/store.ts` (FR-037)
- [ ] T061 [P] [US3] Implement schema migrations in `src/storage/migrate.ts` (FR-049)
- [ ] T062 [US3] Implement the scores store (top 10 per list, ranking, personal best) in `src/storage/scores.ts` (depends on T060) (FR-034)
- [ ] T063 [US3] Implement the settings store with validation and key-swap in `src/storage/settings.ts` (depends on T060) (FR-035)
- [ ] T064 [US3] Add the 3-second countdown before play and after resume, pause and resume, and one-action restart to `src/app/loop.ts` (game time never advances during pause or countdown) (FR-032, FR-033)
- [ ] T065 [P] [US3] Create the Pause screen (resume, restart, settings, and quit; quitting Marathon, Sprint or Ultra asks for confirmation because the game is then discarded) in `src/ui/screens/pause.ts` (FR-025, FR-031, FR-033)
- [ ] T066 [P] [US3] Create the Results screen (score, lines, time, level, personal-best marker, restart, home) in `src/ui/screens/results.ts` (FR-025, FR-033)
- [ ] T067 [P] [US3] Create the High Scores screen with a tab or list per ranking list in `src/ui/screens/high-scores.ts` (FR-031, FR-034)
- [ ] T068 [US3] Create the Settings screen with key remapping, DAS and ARR, and reset scores and settings with confirmation in `src/ui/screens/settings.ts` (FR-027, FR-036)
- [ ] T069 [P] [US3] Create the How to Play screen in `src/ui/screens/how-to-play.ts` (FR-031)
- [ ] T070 [US3] Extend the Home screen to select all four modes (Marathon options kept) in `src/ui/screens/home.ts` (FR-021, FR-022, FR-023, FR-024, FR-031)
- [ ] T071 [US3] Show the Sprint timer to the millisecond, the Ultra countdown and mode-specific HUD in `src/ui/screens/game.ts` (FR-022, FR-023)
- [ ] T072 [US3] Feed saved key map, DAS and ARR into `src/input/keyboard.ts` and game creation in `src/app/main.ts` (FR-027, FR-035)
- [ ] T073 [US3] Save finished games to the scores store and show the personal-best marker, including Sprint completed-only, saving a Zen quit, and discarding a quit from any other mode with no score and no Results screen, in `src/app/main.ts` (FR-025, FR-034)
- [ ] T074 [P] [US3] Write the modes smoke test in `tests/e2e/modes-and-scores.spec.ts`: each mode starts and ends (Ultra and Sprint end by fast-forwarding game time with the e2e test hook, T042), scores and settings persist across reload, the game plays with storage blocked (FR-021, FR-022, FR-023, FR-024, FR-037)

**Checkpoint**: All four modes are playable with results, high scores and settings.

---

## Phase 6: User Story 4 - Play on a phone with touch gestures (Priority: P4)

**Goal**: Drag, flick, tap and swipe gestures control the game on a phone in portrait or landscape, with no browser scroll, zoom, pull-to-refresh or back navigation during play, plus haptics and a first-run overlay.

**Independent Test**: On a real phone, play a full game using only gestures; touch-emulation tests pass and the manual device check in T084 is recorded.

### Tests for User Story 4 (write first, confirm they FAIL) ⚠️

- [ ] T075 [P] [US4] Write failing tests in `tests/unit/gesture-recognizer.test.ts`: pointer sample sequences become taps on left or right half, column-by-column drag steps, slow drag down as held soft drop, fast flick as hard drop, swipe up as hold, sensitivity scaling from 0.5 to 1.5, secondary pointers ignored (thresholds from [research.md](research.md) section 5) (FR-026, FR-028)

### Implementation for User Story 4

- [ ] T076 [P] [US4] Implement the pure gesture recognizer (no DOM) in `src/input/gesture-recognizer.ts` (FR-026, FR-028)
- [ ] T077 [US4] Implement the DOM binding in `src/input/gestures.ts`: Pointer Events on the game surface only, `touch-action: none`, `overscroll-behavior: none`, non-passive `touchmove` `preventDefault` on that surface, emitting `InputEvent`s through the same path as the keyboard (depends on T076) (FR-026, FR-029)
- [ ] T078 [P] [US4] Implement feature-detected vibration (short pulse, silent when unsupported) in `src/haptics/haptics.ts` (FR-040)
- [ ] T079 [US4] Trigger haptics on line clears and hard drops from game events in `src/app/feedback.ts` (FR-040)
- [ ] T080 [US4] Create the responsive layout for 360 px width up to desktop in portrait and landscape and resize the canvas to it in `src/ui/styles/layout.css` and `src/render/canvas.ts` (NFR-002)
- [ ] T081 [US4] Add the corner pause button and attach the gesture adapter to the game surface in `src/ui/screens/game.ts` (FR-026)
- [ ] T082 [US4] Create the dismissable first-run gesture overlay in `src/ui/screens/gesture-overlay.ts` and add gesture sensitivity, haptics on/off and "show gestures again" to `src/ui/screens/settings.ts` (FR-028, FR-030, FR-035, FR-040)
- [ ] T083 [P] [US4] Write touch-emulation tests in `tests/e2e/touch.spec.ts`: gestures produce the expected piece actions, the page does not scroll during play, the first-run overlay shows once and can be reopened (FR-026, FR-029, FR-030)
- [ ] T084 [US4] (Manual, real iPhone and Android phone) Play a full game with gestures, tune the thresholds in `src/input/gesture-recognizer.ts` and defaults in `src/storage/settings.ts` if needed, check whether the edge swipe-back can be triggered in an iOS Safari tab, and record the results in `specs/001-falling-block-puzzle-game/device-notes.md` (FR-029, NFR-002)
- [ ] T085 [US4] If T084 shows the iOS Safari tab swipe-back cannot be prevented, add a short "install for the best experience" hint for iOS tabs on the Home screen in `src/ui/screens/home.ts`; otherwise record "not needed" in `device-notes.md` (FR-029)

**Checkpoint**: The game is fully playable on phones by gestures alone.

---

## Phase 7: User Story 5 - A polished, accessible, installable game (Priority: P5)

**Goal**: Synthesized sound, light and dark themes, accessibility behaviors, and installable offline play with safe updates.

**Independent Test**: Verify sound, theme, reduce-motion and keyboard-only menus; install, go offline and play a full game; automated accessibility, offline and palette tests pass.

### Tests for User Story 5 (write first, confirm they FAIL) ⚠️

- [ ] T086 [P] [US5] Write failing tests in `tests/unit/palette.test.ts`: the 7 pieces have distinct colors and distinct patterns, each tile meets at least 3:1 contrast against the playfield background in light and dark, text tokens meet 4.5:1 (FR-042, FR-046)
- [ ] T087 [P] [US5] Write failing tests in `tests/unit/audio-gate.test.ts`: no audio context is created or resumed before the first user interaction, effects are silent when muted, volume scales the output (FR-041)
- [ ] T088 [P] [US5] Write failing tests in `tests/unit/theme.test.ts`: system theme is followed by default, a manual choice overrides it and is saved, the choice survives reload (FR-039)
- [ ] T089 [P] [US5] Write accessibility smoke tests in `tests/e2e/a11y.spec.ts`: axe scan of Home, Settings, Pause, Results, High Scores and How to Play, keyboard-only reach and operate every control, visible focus (FR-045, FR-046)
- [ ] T090 [P] [US5] Write behavior smoke tests in `tests/e2e/reduce-motion-autopause.spec.ts`: with reduced motion set there is no shake, flash or particle effect, and hiding the page pauses the game (FR-043, FR-044)
- [ ] T091 [P] [US5] Write the offline smoke test in `tests/e2e/offline.spec.ts`: load once online, go offline, reload, and play a short game; also assert every request is same-origin and no cookies are set (FR-047, FR-048)

### Implementation for User Story 5

- [ ] T092 [US5] Implement the synthesized effects (move, rotate, hold, lock, line clear for 1 to 4 lines, T-spin, level up, game over) with an audio context created on the first interaction in `src/audio/sfx.ts` (FR-038, FR-041)
- [ ] T093 [US5] Play effects from game events with the saved mute and volume in `src/app/feedback.ts` (FR-038)
- [ ] T094 [US5] Implement the theme manager (system default, manual override, saved) and apply it to the tokens in `src/ui/theme.ts` (FR-039)
- [ ] T095 [US5] Gate shake, flashes and particle effects on the reduce-motion setting in `src/render/effects.ts` and `src/ui/styles/tokens.css` (FR-043)
- [ ] T096 [US5] Pause automatically on `visibilitychange`, `blur` and `pagehide` and release held keys in `src/app/loop.ts` (FR-044)
- [ ] T097 [US5] Add sound on/off, volume and theme controls to `src/ui/screens/settings.ts` (FR-035, FR-038, FR-039)
- [ ] T098 [US5] Run an accessibility pass over every screen in `src/ui/screens/` and `src/ui/styles/`: labels, headings, focus order, visible focus, announcements, and contrast fixes found by T089 (FR-045, FR-046)
- [ ] T099 [P] [US5] Create original app icons (192, 512 and maskable 512) with `scripts/generate-icons.mjs` from an original design into `public/icons/` and record them in `ASSETS.md` (FR-048, FR-050)
- [ ] T100 [US5] Configure `vite-plugin-pwa` in `vite.config.ts`: full precache, `cacheId` set to the app id, a new service worker that waits until the next launch, scope and `start_url` from the base path (FR-048, FR-049)
- [ ] T101 [US5] Write the manifest per [contracts/pwa-manifest.md](contracts/pwa-manifest.md) and register the service worker in `src/pwa/register.ts` (FR-048, FR-050)
- [ ] T102 [US5] (Manual, one desktop browser, one iPhone and one Android phone) Install the app, play a full game offline, publish a second build and confirm the update appears on next launch with saved data intact, and record the results in `specs/001-falling-block-puzzle-game/device-notes.md` (FR-048, FR-049, SC-003)

**Checkpoint**: All five user stories are complete and independently demonstrable.

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Release readiness, originality, deployment and final validation

- [ ] T103 [P] Create `scripts/check-ip.mjs` that fails if the genre trademark appears anywhere in tracked files or in `dist/` (the term is built from fragments in the script so the script itself does not contain it) (FR-050, SC-009)
- [ ] T104 Add the `check-ip` step to `.github/workflows/ci.yml` (after T001, so it passes) (FR-050)
- [ ] T105 Add a GitHub Pages deploy job to `.github/workflows/ci.yml`: on `main`, build with `BASE_PATH=/<final-repo-name>/` and publish `dist` (after T001) (NFR-008)
- [ ] T106 [P] Finish `README.md` with how to run, build, test and deploy, the controls, the credits and the original-IP statement (N22) (FR-051)
- [ ] T107 [P] Review `ASSETS.md` so every asset (icons, synthesized sound, palette) has a recorded source and licence (FR-050)
- [ ] T108 (Manual) On both reference phones record a 5-minute game and confirm at least 95% of frames take 17 ms or less with none over 50 ms; measure first load under 2 s with the Chrome DevTools "Slow 4G" preset (about 1.6 Mbps down, 150 ms round trip; confirm against current DevTools values) and an empty cache; confirm the total download is under 500 KB (`npm run check-size`); record the reference phone models and results in `specs/001-falling-block-puzzle-game/device-notes.md` (NFR-004, NFR-007, SC-004, SC-007)
- [ ] T109 (Manual) Run the full browser matrix from the spec (desktop Chrome, Edge, Firefox and Safari; iOS Safari, Android Chrome, Samsung Internet) through all four modes (NFR-001, SC-001)
- [ ] T110 Code cleanup and refactoring across `src/`, including a check that `src/core` has no imports from other folders (NFR-009)
- [ ] T111 Walk through every step of [quickstart.md](quickstart.md) and tick each manual item (SC-008)
- [ ] T112 Final Constitution Check: confirm each principle in `.specify/memory/constitution.md` against the finished work and update the post-design check in [plan.md](plan.md) if anything drifted (infrastructure)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: no dependencies
- **Foundational (Phase 2)**: depends on Setup; blocks all user stories
- **User Stories (Phases 3 to 7)**: depend on Foundational and are cumulative. Recommended order is P1 to P5.
- **Polish (Phase 8)**: depends on the user stories you want to ship. T001 gates T104, T105 and any public push.

### User Story Dependencies

- **US1 (P1)**: starts after Foundational. No other story required. This is the MVP.
- **US2 (P2)**: extends `src/core/game.ts` and the renderer from US1.
- **US3 (P3)**: needs US1 (core and screens); results and rankings are only meaningful with US2 scoring. Storage tasks T060 to T063 and their tests have no dependency on US2 and can start early.
- **US4 (P4)**: needs US1 (game screen, input path). Can run in parallel with US2 and US3 after US1, but the Settings controls in T082 need the Settings screen from US3 (T068).
- **US5 (P5)**: sound, theme and PWA tasks only need US1, but the accessibility pass (T098) and the settings controls (T097) need the screens from US3 and US4.

### Within Each User Story

- Tests first, and they must fail before implementation
- Core rules (`src/core`) before the renderer, input and screens that use them
- `src/core/game.ts`, `src/render/canvas.ts`, `src/app/main.ts`, `src/app/loop.ts`, `src/app/feedback.ts` and `src/ui/screens/settings.ts` are edited by several stories, so those tasks run in order rather than in parallel

### Parallel Opportunities

- Setup tasks marked [P] (T003 to T009, T011)
- Foundational tests and implementations marked [P] (T012 to T018)
- All US1 test tasks (T020 to T027) and the four independent core modules (T028 to T031)
- US2 tests (T044 to T047) and `tspin.ts` with `scoring.ts` (T048, T049)
- US3 tests (T054 to T057), storage modules (T060, T061) and the Pause, Results, High Scores and How to Play screens (T065 to T067, T069)
- US5 tests (T086 to T091) and the icon task T099

---

## Parallel Example: User Story 1

```bash
# Write all US1 tests together (each is a separate file):
Task: "Write failing tests in tests/unit/bag.test.ts"
Task: "Write failing tests in tests/unit/srs.test.ts"
Task: "Write failing tests in tests/unit/board.test.ts"
Task: "Write failing tests in tests/unit/gravity.test.ts"
Task: "Write failing tests in tests/unit/lock-delay.test.ts"
Task: "Write failing tests in tests/unit/das-arr.test.ts"
Task: "Write failing tests in tests/unit/marathon.test.ts"
Task: "Write failing replay tests in tests/replay/determinism.test.ts"

# Then implement the independent core modules together:
Task: "Implement the seeded 7-bag in src/core/bag.ts"
Task: "Implement SRS in src/core/srs.ts"
Task: "Implement the playfield in src/core/board.ts"
Task: "Implement the gravity constants in src/core/gravity.ts"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: run `npm test`, `npm run e2e`, and play a Marathon game by hand
5. Demo locally (do not push publicly until T001 is resolved)

### Incremental Delivery

1. Setup and Foundational, then US1 (MVP)
2. Add US2, validate scoring against the worked examples
3. Add US3, validate all four modes and saved data
4. Add US4 and test on real phones early, since gestures are the highest risk (R2)
5. Add US5, then Polish, resolving T001 before any public deploy

### Single Developer Strategy

1. Do the phases in order, finishing each checkpoint before starting the next
2. Start the manual device checks (T084, T102) as soon as a build runs on a phone; thresholds need real-device tuning

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps each task to a user story for traceability
- Tasks marked (Manual) need a person or a real device and cannot be completed by the automated suite
- Verify tests fail before implementing; commit after each task or logical group
- Perfect clear (F19) and 180-degree rotation (F12) are Could items deferred in the plan, so they have no tasks; `src/core/scoring.ts` keeps one extension point for the perfect clear
- Avoid: same-file conflicts between parallel tasks, vague tasks, and cross-story changes that break an earlier story's tests
