# Implementation Plan: Falling-Block Puzzle Game (v1)

**Branch**: none created (the repository is on `main`; `001-falling-block-puzzle-game` is only the spec directory name, not a git branch) | **Date**: 2026-10-26 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/001-falling-block-puzzle-game/spec.md`

## Summary

Build the v1 falling-block puzzle game described in the spec as a static, installable browser app. Rules live in a pure, seeded, DOM-free TypeScript core that advances in fixed 1 ms ticks; a Canvas 2D renderer draws the playfield; menus and HUD are DOM elements so keyboard focus and screen-reader labels work. Input is two independent adapters (keyboard and touch gestures) that both emit the same action events. Sound effects are synthesized at runtime, so there are no audio assets. Scores and settings are saved in a versioned, prefixed `localStorage` wrapper that falls back to memory. A generated service worker precaches the build for offline play. The site deploys to GitHub Pages from CI.

The stack is a plan-time decision (BRD O2). Alternatives that were rejected are in [research.md](research.md); changing the stack before `/speckit-tasks` is cheap.

## Technical Context

**Language/Version**: TypeScript (strict mode), current Node.js LTS for tooling; target evergreen browsers per NFR-001/002

**Primary Dependencies**: Vite (build and dev server); `vite-plugin-pwa` (service worker generation, manifest). No UI framework, no runtime libraries.

**Storage**: Browser `localStorage` behind a wrapper with a versioned schema, key prefix and in-memory fallback. See [contracts/storage-schema.md](contracts/storage-schema.md).

**Testing**: Vitest for unit and replay tests of the core; Playwright for smoke tests (keyboard play, touch emulation, offline). Real-device checks are manual (see [quickstart.md](quickstart.md)).

**Target Platform**: Evergreen browsers listed in the spec (desktop Chrome, Edge, Firefox, Safari; iOS/iPadOS Safari, Android Chrome, Samsung Internet), installed as a PWA or run in a tab

**Project Type**: Single-project static web application (no backend)

**Performance Goals**: 60 fps on a mid-range phone and desktop; input applied on the next frame; first load under 2 s on 4G; total download under 500 KB (NFR-004 to NFR-007)

**Constraints**: Static hosting only on GitHub Pages under a repository sub-path; HTTPS; fully offline after first visit; no personal data, trackers or cookies; core has no DOM dependency

**Scale/Scope**: 1 playfield, 7 piece types, 4 modes, 7 screens, ~51 functional requirements; a single developer; target source size well under 10k lines

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Gate | Pre-research | Post-design |
|---|---|---|---|
| I. Original IP (NON-NEGOTIABLE) | No trademarked name, logo, trade dress or official assets; original visuals; assets sourced and recorded | **Pass for the design** (original palette and patterns, synthesized sound, provisional name "Project Blocks"). **Open violation in the repository**: see Complexity Tracking | Pass for the design; violation remains tracked |
| II. Deterministic, Isolated Game Core | Rules are pure logic; fixed timestep; no DOM, audio, storage or input dependency in the core | Pass | Pass: `src/core` has its own `tsconfig` with no DOM library, so any DOM reference fails the type check; seeded random generator; 1 ms tick |
| III. Test-First for Rules | Automated tests for SRS, T-spin, scoring, 7-bag written before or with the code | Pass | Pass: rule tables and worked scoring examples in [research.md](research.md) and [data-model.md](data-model.md) are the test fixtures |
| IV. Performance and Input Budget | 60 fps, next-frame input, first load under 2 s on 4G, payload under 500 KB | Pass | Pass: no framework, no asset downloads; CI gzip size check; throttled-load check is manual |
| V. Accessible, Private, Resilient by Default | Non-color cues, WCAG 2.2 AA, reduce motion, keyboard menus, auto-pause, no personal data, storage-blocked fallback | Pass | Pass: DOM menus, per-piece patterns, `prefers-reduced-motion`, storage wrapper with fallback, no network calls other than loading the app |
| VI. Scope Discipline | BRD out-of-scope list stays out | Pass | Pass: 180-degree rotation (F12) and perfect clear (F19) are Could items and are deferred; no out-of-scope item is planned |
| Platform: static hosting, browser matrix, 360 px, PWA, offline, update-safe | | Pass | Pass: see [research.md](research.md) sections 8 and 9 |

**Gate result**: the design passes. One repository-level violation of Principle I exists today and blocks the public deploy, not planning. It is recorded below.

## Project Structure

### Documentation (this feature)

```text
specs/001-falling-block-puzzle-game/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
│   ├── core-api.md
│   ├── input-actions.md
│   ├── storage-schema.md
│   └── pwa-manifest.md
├── checklists/
│   └── requirements.md
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)

```text
src/
├── core/                    # Pure rules. Own tsconfig, no DOM lib. No imports from other src/ folders.
│   ├── pieces.ts            # Shapes, spawn states, colors-by-id (no color values)
│   ├── srs.ts               # Rotation states and wall-kick tables
│   ├── board.ts             # Playfield grid, collision, line clears
│   ├── bag.ts               # Seeded 7-bag
│   ├── rng.ts               # Seeded generator
│   ├── gravity.ts           # Per-level gravity table, soft-drop factor
│   ├── tspin.ts             # 3-corner detection, mini vs full
│   ├── scoring.ts           # Clears, T-spins, combo, back-to-back, drops
│   ├── modes.ts             # Marathon, Sprint, Ultra, Zen configs and end rules
│   └── game.ts              # State machine: step(), actions, events, snapshot
├── input/
│   ├── keyboard.ts          # Key events -> actions, DAS/ARR, remapping
│   ├── gestures.ts          # Pointer events -> drag/flick/tap/swipe -> actions
│   └── actions.ts           # Action names shared by both adapters
├── render/
│   ├── canvas.ts            # Playfield, ghost, hold, next queue
│   ├── palette.ts           # Okabe-Ito based colors and per-piece patterns, light/dark
│   └── effects.ts           # Callouts and effects, gated by reduce motion
├── audio/
│   └── sfx.ts               # Web Audio synthesized effects, muted until first interaction
├── haptics/
│   └── haptics.ts           # Feature-detected vibration
├── storage/
│   ├── store.ts             # Prefixed, versioned localStorage wrapper with memory fallback
│   └── migrate.ts           # Schema migrations
├── ui/
│   ├── screens/             # home, game, pause, results, high-scores, settings, how-to-play
│   ├── router.ts            # Screen switching, focus management
│   └── styles/              # CSS custom properties for themes
├── app/
│   ├── loop.ts              # requestAnimationFrame loop, accumulator, auto-pause
│   ├── main.ts              # Wiring
│   └── test-hook.ts         # Fast-forward and scripted input; included only in `vite build --mode e2e`
└── pwa/
    └── register.ts          # Service worker registration

public/
└── icons/                   # Original app icons (generated, recorded in ASSETS.md)

tests/
├── unit/                    # Vitest: srs, tspin, scoring, bag, gravity, modes, store
├── replay/                  # Vitest: seed + timestamped inputs -> expected results (SC-010)
└── e2e/                     # Playwright: keyboard play, touch emulation, offline

scripts/
└── check-size.mjs           # Fails CI if gzipped build exceeds the budget

.github/workflows/
└── ci.yml                   # lint, typecheck (core and app), unit, build, size, e2e, Pages deploy

tsconfig.json                # App config (DOM lib)
tsconfig.core.json           # Core config (no DOM lib)
vite.config.ts               # base path from BASE_PATH, PWA plugin, cacheId
README.md                    # Run, build, test, deploy (N22)
ASSETS.md                    # Source and licence of every asset (Constitution I)
```

**Structure Decision**: One project at the repository root. The `core` folder is the only place rules live and it cannot import from the rest of `src/`; this is checked by its separate `tsconfig` and an ESLint import restriction. The renderer, input, audio and UI layers depend on the core, never the reverse. No backend folder exists because the app is purely static.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| **Principle I (partly resolved; the folder rename remains, and it blocks public deploy)**: the repository directory name and the BRD filename contained the genre trademark, and the BRD body text mentioned it 4 times. Acceptance criterion 6 says it must not appear "anywhere in the repo". | The BRD is the source requirements document and the working directory predates the constitution. | Option (a) was chosen on 2026-10-26: the BRD file was renamed to `blocks-brd.md` and its 4 body mentions were reworded. The remaining step is renaming the working directory, whose current name contains the trademark (the owner does this outside the session) and setting the GitHub repository name to the final game name (BRD O1). Both are tracked as task T001. |
| **Known limitation against FR-029**: in an iOS Safari tab, the system edge swipe-back may not be preventable from a web page. | Browsers own this gesture. | The installed app (standalone display) does not have it. Plan: confirm on a real iPhone, and show the "install for the best experience" hint on iOS if the tab behavior cannot be suppressed. |
| **Known limitation against FR-040**: iOS has no vibration support. | Browser feature gap. | The spec already requires silent skipping where unsupported. |
