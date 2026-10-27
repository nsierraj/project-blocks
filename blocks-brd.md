# Business Requirements Document: Falling-Block Puzzle Game

| | |
|---|---|
| Working title | Project Blocks (placeholder, see Naming) |
| Owner | Amado Sierra |
| Version | 0.1 Draft |
| Date | October 26, 2026 |
| Status | For review |

## 1. Purpose

Build a falling-block puzzle game in the classic falling-block genre that runs in the browser on desktop and mobile. It is a personal portfolio project. The goal is to show clean game-loop engineering, responsive input handling across keyboard and touch, and a polished, installable web app.

## 2. Naming and IP

The genre's best-known title and that game's visual trade dress are trademarks of their owner. This document does not use that title; it describes the genre generically.

- The public name, repo name, page title and app manifest must not contain the genre's trademarked title or a confusingly similar name.
- No official logos, artwork, music arrangements or branding.
- The visual style should be original rather than a copy of any licensed release (see Risk R1).
- A final name is an open item (O1).

## 3. Objectives and Success Criteria

| ID | Objective | Measure of success |
|---|---|---|
| OB1 | Deliver a complete, playable game in four modes | All modes playable start to finish on every supported browser |
| OB2 | Feel responsive on both desktop and mobile | Steady 60 fps on target devices; input acted on in the next frame |
| OB3 | Work offline once installed | Game loads and plays in airplane mode after first visit |
| OB4 | Serve as a portfolio piece | Public repo with README, live link on GitHub Pages, clean commit history |
| OB5 | Meet accessibility basics | All accessibility requirements in section 7.4 pass |

## 4. Scope

### In scope (v1)

- Four game modes: Marathon, Sprint, Ultra, Zen
- Modern mechanics: hold, ghost piece, next queue, SRS rotation with wall kicks, T-spins, combos, back-to-back bonus
- Keyboard controls on desktop, gesture controls on mobile
- Local high scores and settings stored in the browser
- Sound effects, light and dark themes, haptics on supported phones
- Installable PWA with offline play
- Hosting on GitHub Pages

### Out of scope (v1)

- Online or global leaderboards, accounts, sign-in
- Multiplayer or versus modes
- Background music
- On-screen button controls
- Gamepad support
- Monetization, ads, analytics that track players
- Native app store builds

## 5. Stakeholders and Users

| Role | Who | Interest |
|---|---|---|
| Owner, developer | Amado Sierra | Builds and maintains the game; uses it as a portfolio piece |
| Reviewers | Hiring managers, peers | Judge code quality, UX and delivery |
| Players | Casual and experienced genre players | Fast, fair, good-feeling gameplay |

**Player profiles**

- **Casual player on a phone.** Plays short sessions, needs gestures that work without reading instructions.
- **Experienced player on desktop.** Expects guideline-standard rotation, tunable key repeat (DAS/ARR), and accurate Sprint timing.

## 6. Functional Requirements

Priority: **M** = Must, **S** = Should, **C** = Could.

### 6.1 Playfield and pieces

| ID | Requirement | Pri |
|---|---|---|
| F1 | Playfield is 10 columns by 20 visible rows, with hidden rows above for spawning | M |
| F2 | Seven piece types (I, O, T, S, Z, J, L), each with its own color and a second visual cue (pattern or shape mark) | M |
| F3 | Pieces are generated with a 7-bag randomizer: each set of seven contains one of every piece, shuffled | M |
| F4 | Pieces spawn at the top center in a fixed orientation | M |
| F5 | Game over (top-out) when a new piece can't spawn without overlapping blocks, except in Zen | M |

### 6.2 Movement and rotation

| ID | Requirement | Pri |
|---|---|---|
| F6 | Move left and right, soft drop, hard drop | M |
| F7 | Rotate clockwise and counterclockwise using the Super Rotation System (SRS), including standard wall-kick tables for I and for J/L/S/T/Z | M |
| F8 | Lock delay of 0.5 s after a piece touches down; moving or rotating resets it, up to 15 resets per piece | M |
| F9 | Ghost piece shows where the active piece will land | M |
| F10 | Hold slot: swap the active piece into hold once per piece drop | M |
| F11 | Next queue shows the upcoming 5 pieces | M |
| F12 | 180-degree rotation | C |

### 6.3 Line clears and scoring

| ID | Requirement | Pri |
|---|---|---|
| F13 | Full rows clear and rows above shift down | M |
| F14 | Scoring per clear, multiplied by current level: Single 100, Double 300, Triple 500, Quad (4 lines) 800 | M |
| F15 | T-spin detection using the 3-corner rule with mini vs full distinction. Scores × level: T-spin mini 100, T-spin 400, mini single 200, T-spin single 800, double 1200, triple 1600 | M |
| F16 | Back-to-back bonus: ×1.5 on consecutive "difficult" clears (Quads and T-spin clears) | M |
| F17 | Combo bonus: 50 × combo count × level for consecutive piece placements that clear lines | M |
| F18 | Drop points: soft drop 1 per cell, hard drop 2 per cell | M |
| F19 | Perfect clear (empty board) bonus | C |
| F20 | On-screen callouts for T-spins, Quads, back-to-back and combos | S |

### 6.4 Game modes

| ID | Mode | Rules | Pri |
|---|---|---|---|
| F21 | Marathon | Starts at a chosen level (1–15). Level rises every 10 lines. Gravity speeds up by level using the guideline curve. Default ends at 150 lines; an "endless" toggle removes the cap | M |
| F22 | Sprint | Clear 40 lines as fast as possible. Timer shown to the millisecond. Fixed gravity | M |
| F23 | Ultra | Score as much as possible in 3 minutes. Countdown timer | M |
| F24 | Zen | No gravity, no timer, no game over. Pieces fall only on player input. If the stack reaches the top, the board clears and play continues | M |
| F25 | Each mode has its own results screen showing score, lines, time, level, and a personal-best marker | M |

### 6.5 Controls

**Desktop (default keys, all remappable)**

| Action | Key |
|---|---|
| Move left / right | ← / → |
| Soft drop | ↓ |
| Hard drop | Space |
| Rotate clockwise | ↑ or X |
| Rotate counterclockwise | Z |
| Hold | C or Shift |
| Pause | Esc or P |

**Mobile (gestures)**

| Action | Gesture |
|---|---|
| Move left / right | Drag horizontally; piece follows the finger column by column |
| Soft drop | Drag down slowly |
| Hard drop | Fast flick down |
| Rotate clockwise | Tap right half of the screen |
| Rotate counterclockwise | Tap left half of the screen |
| Hold | Swipe up |
| Pause | Pause button in the corner |

| ID | Requirement | Pri |
|---|---|---|
| F26 | Keyboard and gesture mappings as above | M |
| F27 | Adjustable DAS (delayed auto shift) and ARR (auto repeat rate) on desktop | M |
| F28 | Adjustable gesture sensitivity on mobile | S |
| F29 | Gestures must not trigger browser scroll, zoom, pull-to-refresh or back navigation during play | M |
| F30 | First-run overlay on mobile showing the gestures, dismissable and re-openable from Settings | S |

### 6.6 Game flow and screens

| ID | Requirement | Pri |
|---|---|---|
| F31 | Screens: Home (mode select), Game, Pause, Results, High Scores, Settings, How to Play | M |
| F32 | 3-second countdown before play starts and after resuming from pause | S |
| F33 | Restart from Pause and Results with one action | M |

### 6.7 Data and settings

| ID | Requirement | Pri |
|---|---|---|
| F34 | Top 10 scores per mode stored locally (score or time, lines, date) | M |
| F35 | Settings stored locally: key mapping, DAS/ARR, theme, sound on/off and volume, haptics on/off, gesture sensitivity | M |
| F36 | Option to reset all scores and settings, with confirmation | S |
| F37 | Game runs normally if browser storage is blocked; scores just aren't saved | M |

### 6.8 Audio, visuals and feedback

| ID | Requirement | Pri |
|---|---|---|
| F38 | Sound effects for move, rotate, hold, lock, line clear (by size), T-spin, level up, game over. Original or royalty-free | M |
| F39 | Light and dark themes. Follows the system setting by default, with a manual toggle | M |
| F40 | Haptic feedback on line clears and hard drops on devices that support the Vibration API. Silently skipped where unsupported (including iOS Safari) | S |
| F41 | Sound starts muted or waits for first user interaction, per browser autoplay rules | M |

## 7. Non-Functional Requirements

### 7.1 Platform and browser support

| ID | Requirement |
|---|---|
| N1 | Latest two versions of Chrome, Edge, Firefox and Safari on desktop |
| N2 | Latest two versions of Safari on iOS/iPadOS, Chrome on Android, Samsung Internet |
| N3 | Layout works from 360 px phone width up to desktop, in portrait and landscape. Portrait is the primary mobile layout |
| N4 | No plugins or installs required to play |

### 7.2 Performance

| ID | Requirement |
|---|---|
| N5 | Steady 60 fps on a mid-range phone from the last three years and on desktop |
| N6 | Game logic runs on a fixed timestep, independent of display refresh rate (60, 90, 120 Hz screens play at the same speed) |
| N7 | Input is applied on the next frame after the event |
| N8 | First load under 2 seconds on a typical 4G connection; total download size kept small (target under 500 KB) |

### 7.3 Offline and installability

| ID | Requirement |
|---|---|
| N9 | Installable as a PWA (web app manifest, icons, service worker) on desktop and mobile |
| N10 | Fully playable offline after the first visit |
| N11 | Updates are picked up on next launch without breaking saved scores or settings |

### 7.4 Accessibility

| ID | Requirement |
|---|---|
| N12 | Colorblind-safe palette; each piece also has a non-color cue so pieces are distinguishable in grayscale |
| N13 | All keys remappable; DAS/ARR adjustable (F27) |
| N14 | Honors the OS "reduce motion" setting: no screen shake, flashes or particle bursts when set |
| N15 | Game pauses automatically when the tab loses focus or the app is backgrounded |
| N16 | Menus fully usable by keyboard, with visible focus states and screen-reader labels |
| N17 | Text and UI contrast meets WCAG 2.2 AA |

### 7.5 Privacy and security

| ID | Requirement |
|---|---|
| N18 | No personal data collected; no third-party trackers or cookies |
| N19 | Served over HTTPS (GitHub Pages default) |

### 7.6 Maintainability

| ID | Requirement |
|---|---|
| N20 | Game rules (rotation, scoring, randomizer) separated from rendering and input, so they can be unit tested |
| N21 | Automated tests for SRS kicks, T-spin detection, scoring and the 7-bag randomizer |
| N22 | README covering how to run, build, test and deploy |

## 8. Constraints and Assumptions

**Constraints**

- Static hosting only (GitHub Pages). No server-side code.
- Single developer.
- Technology stack is not fixed by this document; it is chosen during design, within the constraints above.

**Assumptions**

- Players accept that scores live on one device and browser.
- Guideline-style rules (SRS, 7-bag, lock delay) are the expected baseline for experienced players.

## 9. Risks

| ID | Risk | Impact | Mitigation |
|---|---|---|---|
| R1 | Trademark or trade-dress claim. Courts have treated the look of the game (piece appearance, board, ghost and next-piece display) as protectable, not only the name | High if published widely | Original name and visual style; no official assets; keep it a non-commercial portfolio piece; review before any wider promotion |
| R2 | Mobile gestures misread (a tap read as a drag, flick read as soft drop) | High | Tunable thresholds, testing on real devices, sensitivity setting |
| R3 | Browser gestures interfere (scroll, pull-to-refresh, iOS swipe-back) | Medium | Disable touch defaults on the game surface; test on iOS and Android |
| R4 | Timing drift on high-refresh screens or when the tab is throttled | Medium | Fixed-timestep loop (N6), auto-pause on blur (N15) |
| R5 | iOS PWA limits (storage eviction, no vibration) | Low | Graceful fallback (F37, F40) |
| R6 | Scope creep with no deadline | Medium | Hold the out-of-scope list; ship v1 by phase |

## 10. Open Items

| ID | Item |
|---|---|
| O1 | Final public name |
| O2 | Technology stack |
| O3 | Visual style direction (block style, palette, background) |
| O4 | Sound source: create, or pick a royalty-free pack |
| O5 | Target dates, if any |

## 11. Delivery Phases

No deadline is set. Work is sequenced so each phase ends with something playable.

| Phase | Content |
|---|---|
| 1. Core engine | Board, pieces, 7-bag, SRS, lock delay, line clears, keyboard input, Marathon |
| 2. Full mechanics | Hold, ghost, next queue, scoring with T-spins, combos, back-to-back |
| 3. Modes and screens | Sprint, Ultra, Zen, results, high scores, settings |
| 4. Mobile | Gesture controls, responsive layout, haptics |
| 5. Polish | Sound, themes, accessibility, PWA and offline |
| 6. Release | Testing on target browsers, README, GitHub Pages deploy |

## 12. Acceptance Criteria

v1 is complete when:

1. All four modes play start to finish on every browser in N1 and N2.
2. Automated tests for rotation, T-spins, scoring and randomizer pass.
3. The game installs and plays offline on one desktop browser, one iPhone and one Android phone.
4. 60 fps holds during play on a mid-range phone.
5. Every Must requirement in sections 6 and 7 is met.
6. No trademarked genre title, logo or official asset appears anywhere in the shipped game or repo.

## 13. Glossary

| Term | Meaning |
|---|---|
| 7-bag | Randomizer that deals all seven pieces in shuffled order before repeating |
| ARR | Auto repeat rate: how fast a held direction key repeats after DAS |
| Back-to-back | Two or more difficult clears in a row with no easy clear between them |
| DAS | Delayed auto shift: wait before a held key starts repeating |
| Ghost piece | Outline showing where the active piece will land |
| Lock delay | Time a piece can still move after touching down |
| Quad | Clearing four lines at once |
| SRS | Super Rotation System: the standard rotation and wall-kick rules |
| T-spin | Rotating a T piece into a tight slot, scored as a bonus |
