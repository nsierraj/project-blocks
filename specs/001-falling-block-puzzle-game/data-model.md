# Data Model: Falling-Block Puzzle Game (v1)

Phase 1 output for [plan.md](plan.md). Entities come from the spec's Key Entities; rule values come from [research.md](research.md) section 4. Field types are described in plain terms; the code representation is a task for implementation.

## Piece

| Field | Meaning | Rules |
|---|---|---|
| `type` | One of I, O, T, S, Z, J, L | Seven fixed types |
| `rotation` | 0, R, 2 or L | Changes only by a successful rotation |
| `x`, `y` | Position of the piece origin on the playfield | Must never overlap a locked block or leave the grid |
| `lastAction` | The last successful action: move, rotate, soft drop, gravity | Needed for T-spin detection (a T-spin requires `rotate`) |
| `lastKickIndex` | Kick test number used by the last rotation (1 to 5) | Test 5 forces a full T-spin |
| `lockTimerMs` | Time left before lock; absent while falling | Starts at 500 on touchdown |
| `lockResetsUsed` | Resets used (0 to 15) | Never refreshes for the same piece |

Static data per type: shape in each rotation state, spawn columns, color id and pattern id (no color values in the core).

## Playfield

| Field | Meaning | Rules |
|---|---|---|
| `cells` | 10 columns by 40 rows; each cell empty or holds a piece type | Rows 0 to 19 visible, 20 to 39 hidden |
| `visibleRows` | 20 | Fixed |

Operations: collision test, lock a piece, find and clear full rows (rows above shift down), test for an empty board. A row is full when all 10 cells are filled.

## Piece Queue

| Field | Meaning | Rules |
|---|---|---|
| `bag` | Remaining pieces in the current shuffled set | Refilled with a new shuffle when empty |
| `upcoming` | Pieces after the active one | Always at least 5 available; shows 5 |
| `rngState` | Seeded generator state | Advances only when a bag is shuffled |

Invariant: every group of 7 pieces from the start of a bag contains one of each type.

## Hold Slot

| Field | Meaning | Rules |
|---|---|---|
| `held` | None or a piece type | |
| `usedThisPiece` | Whether hold was used for the current piece | Set on use, cleared on lock |

## Game Session

| Field | Meaning | Rules |
|---|---|---|
| `mode` | Marathon, Sprint, Ultra or Zen | Fixed for the session |
| `variant` | For Marathon: `150-line` or `endless` | |
| `seed` | Random seed | Saved with the result so a game can be replayed |
| `startLevel` | Starting level | Marathon 1 to 15, otherwise 1 |
| `level` | Current level | `startLevel + floor(lines / 10)`; Sprint fixed at 1 |
| `score` | Current score | Never decreases |
| `lines` | Lines cleared | |
| `elapsedMs` | Game time | Advances only while play runs (not during pause or the countdown) |
| `remainingMs` | Ultra only | `180000 - elapsedMs` |
| `combo` | Combo counter | Starts at -1; see research 4.6 |
| `backToBack` | Whether a difficult clear is pending | See research 4.6 |
| `status` | `playing` or `ended` | |
| `endReason` | `goal`, `time`, `top-out` or `quit` | Set when `status` becomes `ended` |
| `events` | Output queue of things that happened | See [contracts/core-api.md](contracts/core-api.md) |

The pause, countdown and screens are app state, not core state.

### Session state transitions

```text
(new) --seed, mode--> playing
playing --goal reached (Marathon 150 lines, Sprint 40 lines)--> ended (goal)
playing --180 000 ms (Ultra)--> ended (time)
playing --spawn blocked (not Zen)--> ended (top-out)
playing --spawn blocked (Zen)--> playing (board cleared)
playing --player quits (any mode; Zen's only end)--> ended (quit)
```

The core ends a session on `quit()` in any mode. What the app does next depends on the mode: a Zen quit shows Results and saves a Score Entry; a Marathon, Sprint or Ultra quit asks for confirmation, then discards the game (no Score Entry, no Results screen, back to Home).

### Active piece lifecycle

```text
spawn --> falling --cannot move down--> grounded (lock timer 500 ms)
grounded --move or rotate and resets < 15--> grounded (timer reset, resets + 1)
grounded --move or rotate and resets = 15--> grounded (timer not reset)
grounded --move off ledge--> falling (timer cancelled, resets kept)
grounded --timer ends--> locked
falling or grounded --hard drop--> locked
locked --> clear rows, score, update combo and back-to-back --> spawn next
```

In Zen there is no gravity and no lock timer: the piece stays where it is until a hard drop locks it.

## Game Mode (static configuration)

| Mode | Start level | Ends | Gravity | Ranking key | Timer shown |
|---|---|---|---|---|---|
| Marathon | chosen 1 to 15 | 150 lines (not endless), top-out | Level table | Score | Elapsed |
| Sprint | 1 | 40 lines; top-out is an unranked fail | Level 1 fixed | Time (fastest first) | Elapsed, in milliseconds |
| Ultra | 1 | 180 000 ms, top-out | Level table | Score | Countdown |
| Zen | 1 | Player quits | None | Lines | None |

## Score Entry

| Field | Meaning | Rules |
|---|---|---|
| `list` | Which ranking list: `marathon-150`, `marathon-endless`, `sprint`, `ultra`, `zen` | |
| `score` | Final score | Used for marathon and ultra ranking |
| `timeMs` | Final time | Used for sprint ranking; also shown on all results |
| `lines` | Lines cleared | Used for zen ranking |
| `level` | Final level | Shown on results |
| `date` | ISO date and time the game ended | Used as a tie-breaker (earlier ranks first) |
| `seed` | Game seed | Optional, for replays |

Rules: each list keeps the top 10; a new entry that does not rank is not stored; equal values keep both entries; Sprint stores only completed games.

## Settings

| Field | Meaning | Default / range |
|---|---|---|
| `keyMap` | Action to one or more key codes | See [contracts/input-actions.md](contracts/input-actions.md); assigning a used key swaps actions |
| `dasMs` | Delayed auto shift | 167; 50 to 300 |
| `arrMs` | Auto repeat rate | 33; 0 to 100 |
| `gestureSensitivity` | Scales gesture thresholds | 1.0; 0.5 to 1.5 |
| `theme` | `system`, `light` or `dark` | `system` |
| `soundOn`, `volume` | Effects on/off and volume | on, 0.7 (volume 0 to 1) |
| `hapticsOn` | Vibration on/off | on |
| `gestureOverlaySeen` | Whether the first-run overlay was dismissed | false |
| `marathonStartLevel`, `marathonEndless` | Last-used Marathon options | 1, false |

## Saved data envelope

Everything is stored under the `blocks:` prefix with a `schemaVersion`. See [contracts/storage-schema.md](contracts/storage-schema.md). Unknown, invalid or older data is migrated or replaced by defaults.

## Gesture

| Gesture | Recognised when | Produces |
|---|---|---|
| Tap right half | Short, almost no movement, right of center | Rotate clockwise |
| Tap left half | Same, left of center | Rotate counterclockwise |
| Horizontal drag | Horizontal travel on the game surface | One-shot left/right steps, a column per cell width |
| Slow drag down | Vertical travel past the dead zone, below flick speed | Soft drop held |
| Flick down | Vertical travel at or above the flick speed | Hard drop |
| Swipe up | Long, fast, mostly upward | Hold |

Thresholds are in [research.md](research.md) section 5. Only the primary pointer counts.
