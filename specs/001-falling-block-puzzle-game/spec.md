# Feature Specification: Falling-Block Puzzle Game (v1)

**Feature Branch**: `001-falling-block-puzzle-game` (spec directory name; no git branch was created)

**Created**: 2026-10-26

**Status**: Draft

**Input**: User description: "Use the business requirements document in the repository root (BRD v0.1) as the feature description: a browser-based falling-block puzzle game with four modes, modern mechanics, keyboard and touch-gesture controls, local scores and settings, and an installable offline-capable web app."

## Clarifications

### Session 2026-10-26

- Q: How should a Zen session end, and what should its saved result be ranked by on the High Scores list? → A: The player ends a Zen session by quitting from Pause; Zen results are ranked by lines cleared (score, time and level are still shown on Results).
- Q: In Ultra mode, should the level (and with it the fall speed and score multiplier) rise as the player clears lines? → A: Yes. Ultra starts at level 1 and the level rises every 10 lines on the same speed curve as Marathon.
- Q: What fixed fall speed should Sprint use for the whole game? → A: Level 1 speed (about 1 row per second) for the whole game.
- Q: On a device with both a touchscreen and a keyboard, which controls should the game accept? → A: Both at the same time, on every device, with no mode switching; gestures only count on the game surface.
- Q: Should Marathon games played with the "endless" option be saved to the same High Scores list as the standard 150-line Marathon games? → A: No. Marathon keeps two separate lists, 150-line and endless, each with its own top 10 and personal best.

## User Scenarios & Testing *(mandatory)*

Stories follow the BRD delivery phases (BRD §11). Each story ends with something playable and can be tested on its own. Phase 6 (release) is covered by the Success Criteria rather than a story.

### User Story 1 - Play a Marathon game with the keyboard (Priority: P1)

A player on a desktop browser opens the game, starts Marathon from the home screen and plays a full game: pieces fall into a 10-column playfield, the player moves, rotates and drops them with the keyboard, full rows clear, the level rises, and the game ends when the stack reaches the top or the line goal is met.

**Why this priority**: This is the core engine. Without correct pieces, rotation, lock delay and line clears, nothing else has value. It is the smallest slice that is a real game (BRD phase 1).

**Independent Test**: Start Marathon, play with the default keys until a game over, and confirm pieces spawn, move, rotate, lock, clear rows and end the game as described below.

**Acceptance Scenarios**:

1. **Given** a new game, **When** play starts, **Then** the playfield shows 10 columns by 20 visible rows and a piece appears at the top center in its fixed starting orientation.
2. **Given** a sequence of pieces, **When** any 7 consecutive pieces are dealt from the start of a bag, **Then** each of the seven piece types appears exactly once.
3. **Given** an active piece next to a wall or other blocks, **When** the player rotates it, **Then** the standard wall-kick rules are tried so the piece rotates into the first valid position, and it does not rotate if none is valid.
4. **Given** a piece resting on the stack, **When** 0.5 seconds pass with no movement or rotation, **Then** the piece locks; **and** moving or rotating before then resets the 0.5 second timer, up to 15 resets for that piece.
5. **Given** one or more complete rows, **When** a piece locks, **Then** those rows disappear and the rows above shift down.
6. **Given** Marathon with a chosen starting level from 1 to 15, **When** the player clears 10 lines, **Then** the level rises by one and pieces fall faster.
7. **Given** a new piece that cannot spawn without overlapping existing blocks, **When** it tries to appear, **Then** the game ends.
8. **Given** Marathon in its default setting, **When** the player reaches 150 lines, **Then** the game ends as a completion; **and** with the endless option on, play continues past 150 lines.

---

### User Story 2 - Use full mechanics: hold, ghost, next queue and bonus scoring (Priority: P2)

The player gets the modern aids and scoring depth that experienced players expect: a ghost piece showing where the piece will land, a hold slot, a preview of upcoming pieces, and scoring that rewards T-spins, combos and back-to-back difficult clears, with on-screen callouts.

**Why this priority**: These features turn a basic engine into the guideline-style game the BRD targets and are required for the scoring and test criteria. They build directly on Story 1 (BRD phase 2).

**Independent Test**: Play Marathon and perform each scoring event (single to Quad, each T-spin type, a combo, back-to-back, soft and hard drops) and confirm the points and callouts match the scoring rules in the requirements.

**Acceptance Scenarios**:

1. **Given** an active piece, **When** it is falling, **Then** a ghost outline shows the exact position where it would land.
2. **Given** an active piece, **When** the player uses hold, **Then** it swaps with the held piece (or the next piece if hold is empty); **and** a second hold before the next piece locks has no effect.
3. **Given** a game in progress, **When** the player looks at the queue, **Then** the next 5 pieces are shown in order.
4. **Given** level 3, **When** the player clears a Double, **Then** the score increases by 900 (300 × level).
5. **Given** a T piece rotated into a tight slot that meets the 3-corner rule, **When** it locks, **Then** the game distinguishes a mini from a full T-spin and awards the matching points.
6. **Given** two difficult clears (Quad or T-spin clear) in a row with no easy clear between them, **When** the second clears, **Then** its points are multiplied by 1.5 and a back-to-back callout is shown.
7. **Given** consecutive piece placements that each clear lines, **When** the player keeps clearing, **Then** each clear after the first adds 50 × combo count × level and a combo callout is shown.
8. **Given** a soft drop or a hard drop, **When** the piece moves down, **Then** the score rises by 1 point per cell for soft drop and 2 points per cell for hard drop.

---

### User Story 3 - Play all four modes with results, high scores and settings (Priority: P3)

The player picks from Marathon, Sprint, Ultra and Zen, sees a results screen at the end of each game, can browse the top 10 scores per mode, and can change key bindings, key-repeat speed and other settings, all saved on the device.

**Why this priority**: It completes the product's four-mode promise (OB1) and the game flow around it (BRD phase 3).

**Independent Test**: Play one short game in each mode, check the results screen, reopen the game to confirm scores and settings persisted, and block storage to confirm the game still runs.

**Acceptance Scenarios**:

1. **Given** Sprint mode, **When** the player clears 40 lines, **Then** the game ends and the results show the elapsed time to the millisecond; **and** the timer runs live in milliseconds during play at a fixed fall speed of level 1 (about 1 row per second) for the whole game.
2. **Given** Ultra mode, **When** 3 minutes elapse, **Then** a visible countdown reaches zero and the game ends with the final score; **and** the game starts at level 1, and the level (with fall speed and score multiplier) rises every 10 lines cleared, as in Marathon.
3. **Given** Zen mode, **When** the game starts, **Then** there is no timer, no automatic falling and no game over; pieces move down only on player input; **and** if the stack reaches the top, the board clears and play continues; **and** the session ends only when the player quits from Pause, which shows the Results screen.
4. **Given** any finished game, **When** the results screen appears, **Then** it shows score, lines, time and level, and marks a new personal best when one is set.
5. **Given** a finished game, **When** the player chooses restart from Pause or Results, **Then** a new game of the same mode starts with one action.
6. **Given** a score that belongs in a mode's top 10, **When** the game ends, **Then** it is saved with its score or time, lines and date, and appears on the High Scores screen for that mode; **and** Marathon and Ultra rank by score (highest first), Sprint by time (fastest first), and Zen by lines cleared (highest first); **and** Marathon keeps two separate lists, one for 150-line games and one for endless games, each with its own top 10 and personal best.
7. **Given** the Settings screen, **When** the player changes key bindings, key-repeat delay and rate, theme, sound, haptics or gesture sensitivity, **Then** the change takes effect and is remembered the next time the game opens.
8. **Given** the player chooses to reset scores and settings, **When** they confirm, **Then** all saved scores and settings return to defaults; **and** without confirmation nothing is erased.
9. **Given** browser storage is blocked, **When** the player plays, **Then** the game runs normally and scores are simply not saved.
10. **Given** the app, **When** the player navigates, **Then** Home, Game, Pause, Results, High Scores, Settings and How to Play screens are all reachable.
11. **Given** a Marathon, Sprint or Ultra game in progress, **When** the player quits from Pause and confirms, **Then** the game ends without saving a score and Home appears; **and** declining the confirmation returns to Pause.

---

### User Story 4 - Play on a phone with touch gestures (Priority: P4)

A casual player on a phone plays in portrait with gestures: drag to move, slow drag down to soft drop, flick down to hard drop, tap the screen halves to rotate, swipe up to hold. The page never scrolls, zooms or navigates away during play.

**Why this priority**: Mobile is a primary target (OB2), and gestures are the riskiest interaction to get right, but they build on a working game (BRD phase 4).

**Independent Test**: On a real phone, play a full game using only gestures and confirm each gesture does what is listed with no browser scroll, zoom, pull-to-refresh or back navigation.

**Acceptance Scenarios**:

1. **Given** a phone in portrait, **When** the player drags horizontally, **Then** the piece follows the finger column by column.
2. **Given** an active piece, **When** the player drags down slowly, **Then** it soft drops; **and** a fast downward flick hard drops it.
3. **Given** an active piece, **When** the player taps the right half of the screen, **Then** it rotates clockwise; **and** tapping the left half rotates it counterclockwise.
4. **Given** an active piece, **When** the player swipes up, **Then** hold is used.
5. **Given** play in progress, **When** the player uses any of these gestures, **Then** the browser does not scroll, zoom, pull to refresh or navigate back.
6. **Given** a first run on a phone, **When** the game opens, **Then** an overlay explains the gestures, can be dismissed, and can be reopened from Settings.
7. **Given** the sensitivity setting, **When** the player changes it, **Then** the distances and speeds that distinguish a tap, drag and flick change accordingly.
8. **Given** a phone that supports vibration, **When** lines clear or a hard drop lands and haptics are on, **Then** the device vibrates; **and** where vibration is unsupported nothing happens and no error shows.
9. **Given** a screen width from 360 px up, **When** the player rotates between portrait and landscape, **Then** the layout stays usable in both, with portrait as the primary phone layout.

---

### User Story 5 - Enjoy a polished, accessible, installable game (Priority: P5)

The player hears sound effects, can use light or dark themes, can rely on accessibility features, and can install the game and play it offline.

**Why this priority**: Polish and accessibility are explicit objectives (OB3, OB5) but depend on the game and screens already existing (BRD phase 5).

**Independent Test**: Verify sound, theme, reduce-motion and keyboard-only menu behavior; install the game, switch to airplane mode and play a full game.

**Acceptance Scenarios**:

1. **Given** the game just loaded, **When** the player has not yet interacted, **Then** sound is muted or waits until the first interaction; **and** afterward effects play for move, rotate, hold, lock, line clear (by size), T-spin, level up and game over, at the chosen volume.
2. **Given** the system is set to dark or light, **When** the game opens, **Then** it uses the same theme; **and** a manual toggle overrides it.
3. **Given** the operating system's reduce-motion setting is on, **When** play happens, **Then** there is no screen shake, flash or particle burst.
4. **Given** play in progress, **When** the tab loses focus or the app is backgrounded, **Then** the game pauses automatically.
5. **Given** any menu, **When** the player uses only the keyboard, **Then** every control is reachable and operable, with a visible focus state and a label a screen reader can read.
6. **Given** the seven piece types, **When** viewed in grayscale, **Then** each is still distinguishable through a pattern or shape mark, and the color palette is colorblind-safe.
7. **Given** a first visit online, **When** the player installs the game on desktop, iPhone or Android and then goes offline, **Then** the game opens and plays fully.
8. **Given** a new version is published, **When** the player next launches the game, **Then** the update is picked up and saved scores and settings remain intact.

---

### Edge Cases

- A new piece's spawn position is blocked, including right after using hold: the game ends rather than overlapping blocks (except in Zen, which clears the board instead).
- A rotation has no valid wall-kick position: the piece stays where it is and the lock timer is not reset.
- A piece reaches the 15-reset limit while still sliding or rotating: it locks at the end of the current 0.5 second delay without further resets.
- A T-spin that clears no lines, a T-spin mini, and a T-spin that clears one, two or three lines are all scored as separate cases.
- A combo is broken by a placement that clears no lines; back-to-back is broken by a non-difficult line clear but not by a placement that clears no lines.
- The player clears the board completely (perfect clear): an optional bonus is awarded if the feature is built.
- Ultra's 3 minutes expire while a piece is falling: the game ends at that moment and the final score is the score at that point.
- The player pauses and resumes: a 3-second countdown runs before play continues, and the game never advances while paused.
- A display runs at 60, 90 or 120 Hz, or the tab is throttled: pieces fall at the same speed on all of them, and a throttled tab pauses.
- Two or more touches at once on a phone: the game responds to the primary gesture and ignores stray touches rather than misfiring.
- A key is held down: movement repeats according to the player's delay and rate settings.
- A device has both a touchscreen and a keyboard (such as a tablet with a keyboard or a touchscreen laptop): keys and gestures both work at the same time during play, and touches on menus act as ordinary taps rather than gestures.
- Two saved scores are equal: both are kept; the earlier entry ranks first.
- Saved data is damaged or from an older version: the game recovers to defaults without crashing.
- Storage is full or blocked part-way through a session: play continues and the player is not shown a blocking error.
- A key binding is set to a key already used by another action: the game warns or swaps rather than leaving two actions on one key.
- Haptics or sound are unsupported or blocked by the browser: the game continues silently.
- The player quits a Marathon, Sprint or Ultra game from Pause: after confirming, the game is discarded with no saved score and no Results screen; only Zen saves on quit.

## Requirements *(mandatory)*

IDs in parentheses cite the BRD. Priority follows the BRD: **M** = Must, **S** = Should, **C** = Could.

### Functional Requirements

**Playfield and pieces**

- **FR-001**: The playfield MUST be 10 columns by 20 visible rows, with hidden rows above for spawning. (F1, M)
- **FR-002**: The game MUST have seven piece types, each with its own color and a second visual cue (pattern or shape mark). (F2, M)
- **FR-003**: Pieces MUST be dealt in shuffled sets of seven that each contain one of every piece type. (F3, M)
- **FR-004**: Pieces MUST spawn at the top center in a fixed orientation. (F4, M)
- **FR-005**: The game MUST end when a new piece cannot spawn without overlapping blocks, except in Zen. (F5, M)

**Movement and rotation**

- **FR-006**: Players MUST be able to move left and right, soft drop and hard drop. (F6, M)
- **FR-007**: Players MUST be able to rotate clockwise and counterclockwise using the standard Super Rotation System, including its standard wall-kick tables for the I piece and for J, L, S, T and Z. (F7, M)
- **FR-008**: A piece MUST lock 0.5 seconds after it touches down; moving or rotating MUST reset the delay, up to 15 resets per piece. This does not apply in Zen, where pieces have no gravity and lock only on hard drop (FR-024). (F8, M)
- **FR-009**: A ghost piece MUST show where the active piece will land. (F9, M)
- **FR-010**: Players MUST be able to swap the active piece with a hold slot once per piece drop. (F10, M)
- **FR-011**: The game MUST show the next 5 upcoming pieces. (F11, M)
- **FR-012**: The game MAY support 180-degree rotation; if built, it MUST follow the same wall-kick behavior as other rotations. (F12, C)

**Line clears and scoring**

- **FR-013**: Complete rows MUST clear and the rows above MUST shift down. (F13, M)
- **FR-014**: Line clears MUST score, multiplied by the current level: Single 100, Double 300, Triple 500, Quad (4 lines) 800. (F14, M)
- **FR-015**: T-spins MUST be detected with the 3-corner rule, separating mini from full, and score (× level): T-spin mini 100, T-spin 400, mini single 200, T-spin single 800, double 1200, triple 1600. (F15, M)
- **FR-016**: Consecutive difficult clears (Quads and T-spin clears) MUST earn a back-to-back bonus of 1.5 times the normal points. (F16, M)
- **FR-017**: Consecutive piece placements that clear lines MUST earn a combo bonus of 50 × combo count × level. (F17, M)
- **FR-018**: Dropping MUST score 1 point per cell for soft drop and 2 points per cell for hard drop. (F18, M)
- **FR-019**: A perfect clear (empty board) MAY award a bonus. (F19, C)
- **FR-020**: The game SHOULD show on-screen callouts for T-spins, Quads, back-to-back and combos. (F20, S)

**Game modes**

- **FR-021**: Marathon MUST start at a chosen level (1 to 15), raise the level every 10 lines, speed up falling by level using the guideline curve, and by default end at 150 lines, with an endless option that removes the cap. (F21, M)
- **FR-022**: Sprint MUST end when 40 lines are cleared, show a timer to the millisecond, and use a fixed fall speed of level 1 (about 1 row per second) for the whole game. (F22, M)
- **FR-023**: Ultra MUST give 3 minutes to score as much as possible, with a countdown timer. Ultra MUST start at level 1 and raise the level every 10 lines on the same speed curve as Marathon. (F23, M)
- **FR-024**: Zen MUST have no automatic falling, no timer and no game over; pieces fall only on player input; if the stack reaches the top, the board MUST clear and play continue. In Zen a piece MUST lock only when the player hard drops it. A Zen session MUST end only when the player quits from Pause, and then show the Results screen. (F24, M)
- **FR-025**: Each mode MUST have a results screen showing score, lines, time, level and a personal-best marker. Quitting an in-progress Marathon, Sprint or Ultra game from Pause MUST ask for confirmation, then end the game without saving a score and return to Home, with no Results screen; only a Zen quit shows Results and is saved. (F25, M)

**Controls**

- **FR-026**: Keyboard controls MUST default to: move with ← and →, soft drop with ↓, hard drop with Space, rotate clockwise with ↑ or X, rotate counterclockwise with Z, hold with C or Shift, pause with Esc or P; and mobile gestures MUST be: horizontal drag to move, slow downward drag to soft drop, fast downward flick to hard drop, tap right half to rotate clockwise, tap left half to rotate counterclockwise, swipe up to hold, and a corner button to pause. Keyboard and gesture controls MUST both be active at the same time on every device, with no mode switching, and gestures MUST only be recognised on the game surface, not on menus. (F26, M)
- **FR-027**: All desktop keys MUST be remappable, and key-repeat delay and rate MUST be adjustable. (F27, N13, M)
- **FR-028**: Gesture sensitivity SHOULD be adjustable on mobile. (F28, S)
- **FR-029**: During play, gestures MUST NOT trigger browser scrolling, zooming, pull-to-refresh or back navigation. (F29, M)
- **FR-030**: A first-run gesture overlay SHOULD appear on mobile, be dismissable, and be reopenable from Settings. (F30, S)

**Game flow and screens**

- **FR-031**: The game MUST provide Home (mode select), Game, Pause, Results, High Scores, Settings and How to Play screens. (F31, M)
- **FR-032**: The game SHOULD show a 3-second countdown before play starts and after resuming from pause. (F32, S)
- **FR-033**: Players MUST be able to restart from Pause and Results with one action. (F33, M)

**Data and settings**

- **FR-034**: The game MUST keep the top 10 scores per mode on the device, each with score or time, lines and date. Marathon and Ultra MUST rank by score (highest first), Sprint by time (fastest first), and Zen by lines cleared (highest first). Marathon MUST keep separate top 10 lists, with separate personal bests, for 150-line games and endless games. (F34, M)
- **FR-035**: The game MUST remember key mapping, key-repeat settings, theme, sound on/off and volume, haptics on/off and gesture sensitivity. (F35, M)
- **FR-036**: The game SHOULD let players reset all scores and settings after confirming. (F36, S)
- **FR-037**: The game MUST run normally if browser storage is blocked; scores are then not saved. (F37, M)

**Audio, visuals and feedback**

- **FR-038**: The game MUST have sound effects for move, rotate, hold, lock, line clear (by size), T-spin, level up and game over, all original or royalty-free. (F38, M)
- **FR-039**: The game MUST offer light and dark themes that follow the system setting by default and can be switched manually. (F39, M)
- **FR-040**: The game SHOULD give haptic feedback on line clears and hard drops on devices that support it, and skip it silently elsewhere. (F40, S)
- **FR-041**: Sound MUST start muted or wait for the first user interaction. (F41, M)

**Accessibility, privacy and installability**

- **FR-042**: The color palette MUST be colorblind-safe and every piece MUST be distinguishable in grayscale. (N12, M)
- **FR-043**: The game MUST honor the operating system's reduce-motion setting by removing screen shake, flashes and particle bursts. (N14, M)
- **FR-044**: The game MUST pause automatically when the tab loses focus or the app is backgrounded. (N15, M)
- **FR-045**: Menus MUST be fully keyboard-usable, with visible focus states and screen-reader labels. (N16, M)
- **FR-046**: Text and interface contrast MUST meet WCAG 2.2 AA. (N17, M)
- **FR-047**: The game MUST NOT collect personal data or use third-party trackers or cookies. (N18, M)
- **FR-048**: The game MUST be installable as an app on desktop and mobile and fully playable offline after the first visit. (N9, N10, M)
- **FR-049**: Updates MUST be picked up on next launch without breaking saved scores or settings. (N11, M)

**Originality and documentation**

- **FR-050**: The public name, repo name, page title and app manifest MUST NOT contain the genre-defining trademark or a confusingly similar name, and no official logos, artwork, music or other assets may appear in the game or repo; visuals MUST follow an original style. (BRD §2, R1, Constitution I, M)
- **FR-051**: The repository MUST include a README covering how to run, build, test and deploy. (N22, M)

### Non-Functional Requirements

- **NFR-001**: The game MUST support the latest two versions of Chrome, Edge, Firefox and Safari on desktop, and the latest two versions of Safari on iOS/iPadOS, Chrome on Android and Samsung Internet. (N1, N2)
- **NFR-002**: The layout MUST work from 360 px width to desktop, in portrait and landscape, with portrait as the primary phone layout. (N3)
- **NFR-003**: The game MUST need no plugins or installs to play in a browser. (N4)
- **NFR-004**: The game MUST hold 60 frames per second on the reference phones and on desktop, meaning that over a 5-minute game at least 95% of frames take 17 ms or less and no frame takes more than 50 ms. The reference phones are one Android phone and one iPhone, each released within the last three years and below the maker's flagship line; the models used are recorded in `device-notes.md`. (N5)
- **NFR-005**: Game speed MUST be identical on 60, 90 and 120 Hz displays. (N6)
- **NFR-006**: Player input MUST be applied on the next frame after the event. (N7)
- **NFR-007**: First load MUST complete in under 2 seconds on a typical 4G connection, measured with the Chrome DevTools "Slow 4G" throttling preset and an empty cache, with a total download target under 500 KB. (N8)
- **NFR-008**: The game MUST be served over HTTPS from static hosting with no server-side code. (N19, BRD §8)
- **NFR-009**: Game rules MUST be separated from display and input so they can be tested alone, and automated tests MUST cover rotation and wall kicks, T-spin detection, scoring and piece-set dealing. (N20, N21, Constitution II, III)

### Key Entities *(include if feature involves data)*

- **Piece**: One of seven shapes, with a color, a second visual cue, a spawn position and orientation, and a current rotation state.
- **Playfield**: The 10-column grid of locked blocks, including hidden spawn rows above the 20 visible rows.
- **Piece Queue**: The ordered upcoming pieces, dealt in shuffled sets of seven, with the next 5 visible to the player.
- **Hold Slot**: Zero or one piece set aside, with a flag showing whether hold was already used for the current piece.
- **Game Session**: One play of one mode, with its mode, level, score, lines cleared, elapsed or remaining time, combo count, back-to-back state and status (playing, paused, ended).
- **Game Mode**: Marathon, Sprint, Ultra or Zen, each with its own goal, fall speed rule and end condition.
- **Score Entry**: A saved result with mode, score or time, lines and date; each mode keeps the top 10, ranked by score (Marathon, Ultra), time (Sprint) or lines cleared (Zen); Marathon keeps separate lists for 150-line and endless games.
- **Settings**: Key mapping, key-repeat delay and rate, theme, sound on/off and volume, haptics on/off, gesture sensitivity, and whether the gesture overlay was seen.
- **Gesture**: A recognised touch action (drag, slow drag down, flick down, tap left or right half, swipe up) mapped to a game action.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Every one of the four modes can be played from start to finish, with no blocking defect, on every browser in the supported list. (OB1, acceptance criterion 1)
- **SC-002**: Automated tests for rotation and wall kicks, T-spin detection, scoring and piece-set dealing all pass. (acceptance criterion 2)
- **SC-003**: The game installs and plays a full game in airplane mode on at least one desktop browser, one iPhone and one Android phone. (OB3, acceptance criterion 3)
- **SC-004**: Play meets the frame-rate rule in NFR-004 on both reference phones during a full game. (OB2, acceptance criterion 4)
- **SC-005**: A new player on a phone completes their first line clear using only gestures, without reading instructions, within their first game.
- **SC-006**: A game played on a 60 Hz and a 120 Hz screen shows the same fall speed, with no more than 1% difference in time to clear the same number of rows at the same level.
- **SC-007**: First load completes in under 2 seconds under the throttling defined in NFR-007 and the total download stays under 500 KB.
- **SC-008**: Every Must requirement in this specification is met and every accessibility requirement (FR-042 to FR-046) passes. (OB5, acceptance criterion 5)
- **SC-009**: No trademarked name, logo or official asset appears anywhere in the shipped game or repository. (acceptance criterion 6)
- **SC-010**: Sprint results are timed to the millisecond and repeat runs of the same input sequence give the same time.

## Assumptions

- The technology stack is not part of this specification; it is chosen during planning within the constraints above (BRD O2).
- The public name is provisional ("Project Blocks") until a final name is chosen (BRD O1); the final name cannot contain the genre-defining trademark.
- Visual style direction (block style, palette, background) is open (BRD O3); it only has to be original, colorblind-safe and meet contrast requirements.
- Sound is either created or taken from a royalty-free pack (BRD O4); the source must be recorded.
- No target dates are set (BRD O5); phases are ordered by dependency, not by deadline.
- Scores and settings live on one device and browser; players accept that they do not sync (BRD §8).
- Guideline-style rules (SRS, 7-bag, lock delay) are the expected baseline for experienced players (BRD §8).
- The game is hosted as a static site with no server-side code, no accounts and no online features (BRD §8).
- Out of scope for v1: online or global leaderboards, accounts, multiplayer or versus modes, background music, on-screen button controls, gamepad support, monetization, ads, tracking analytics, and native app store builds (BRD §4).
- Perfect clear bonus (FR-019) and 180-degree rotation (FR-012) are optional "Could" items and do not block v1.
- There is a single developer and the project is a non-commercial portfolio piece.
