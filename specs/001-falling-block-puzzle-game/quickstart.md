# Quickstart: Validating the Falling-Block Puzzle Game

How to run and prove the feature works end to end. **No code exists yet.** Every command below becomes available only after the project is scaffolded by the first tasks from `/speckit-tasks`; script names are the plan's intent and may be adjusted when scaffolding.

Related: [plan.md](plan.md) · [data-model.md](data-model.md) · [contracts/](contracts/) · [research.md](research.md)

## Prerequisites

- Current Node.js LTS and npm
- A browser from the supported list; for phone checks, a real iPhone and a real Android phone
- A GitHub repository with Pages enabled (for the deploy check), named with the final game name, not the working directory name (see plan Complexity Tracking)

## Commands (available after scaffolding)

| Purpose | Command |
|---|---|
| Install | `npm install` |
| Dev server | `npm run dev` |
| Typecheck app and core | `npm run typecheck` (runs both `tsconfig.json` and `tsconfig.core.json`) |
| Lint (includes core import restriction) | `npm run lint` |
| Unit and replay tests | `npm test` |
| Production build | `BASE_PATH=/<repo-name>/ npm run build` |
| Size budget check | `npm run check-size` (fails if the gzipped build exceeds 500 KB) |
| Preview the build | `npm run preview` |
| Browser smoke tests | `npm run e2e` (builds with `--mode e2e` for the game tests; the offline test uses the normal production build) |

## Automated checks (run in CI)

| Check | Proves | Spec reference |
|---|---|---|
| Core typecheck with no DOM library | Rules do not touch the DOM | NFR-009, Constitution II |
| Unit tests: SRS tables, T-spin, scoring (worked examples in research 4.6), 7-bag, gravity table, modes, storage | Rules are correct | SC-002, FR-007, FR-014 to FR-017 |
| Replay tests: same seed and inputs give identical score, lines and time; 60, 90 and 120 Hz frame schedules agree | Determinism, refresh independence | SC-006, SC-010, NFR-005 |
| Size check | Download budget | SC-007, NFR-007 |
| Playwright keyboard smoke | A Marathon game starts, pieces move, lines clear, game ends | User Story 1 |
| Playwright touch emulation | Gestures produce actions; page does not scroll | User Story 4 (partial) |
| Playwright offline reload | App loads and plays after going offline | User Story 5, SC-003 (partial) |
| axe scan of menu screens | Labels, focus, contrast | FR-045, FR-046 (partial) |

Playwright's WebKit is not iOS Safari. Treat it as a smoke test only.

## Manual validation (real devices or throttling)

Run these before declaring v1 complete.

1. **User Story 1, desktop**: run `npm run dev`, start Marathon at level 1, play with the default keys. Confirm spawn, SRS kicks at a wall, 0.5 s lock delay with 15 resets, line clears, level up at 10 lines, game over on top-out, and completion at 150 lines.
2. **User Story 2**: confirm ghost, hold (once per piece), 5-piece next queue, and the scoring and callouts for a T-spin, a Quad, a back-to-back and a combo.
3. **User Story 3**: play a short game in each mode. Confirm Sprint shows milliseconds, Ultra counts down 3 minutes, Zen has no gravity and ends only from Pause, the Results screen, and that scores and settings survive a reload. Block site data in the browser and confirm the game still plays.
4. **User Story 4, real phones (iPhone and Android)**: play a full game using only gestures. Confirm no scroll, zoom, pull-to-refresh or back navigation during play; on iOS Safari, note whether the edge swipe-back can be triggered in a tab and whether the installed app avoids it. Tune gesture thresholds with the sensitivity setting.
5. **User Story 5**: confirm sound starts muted until the first interaction, light/dark themes follow the system and the toggle overrides it, reduce motion removes shake, flashes and particles, a keyboard-only pass through every menu, and a screen reader pass.
6. **Install and offline (SC-003)**: install on one desktop browser, one iPhone (Add to Home Screen) and one Android phone; go to airplane mode; play a full game. Publish a new build and confirm the update appears on next launch with saved data intact.
7. **Performance (SC-004, SC-007)**: on both reference phones, record a 5-minute game (remote DevTools performance recording) and confirm at least 95% of frames take 17 ms or less with none over 50 ms; measure first load under 2 s with the DevTools "Slow 4G" preset and an empty cache, with the total download under 500 KB.
8. **Originality (SC-009)**: search the repository for the genre trademark and scan the shipped game and manifest for official names, logos or assets. This will fail until the repository naming item in the plan is resolved.

## Expected outcome

Every automated check passes in CI, every manual step above is ticked on the target browsers and devices, and each of the five user stories can be demonstrated on its own.
