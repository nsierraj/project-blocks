# Contract: Input Actions and Mappings

Both input adapters (keyboard and gestures) produce the same action events for the core ([core-api.md](core-api.md)). Both are active at the same time on every device. Gestures are recognised only on the game surface.

## Actions

| Action | Kind | Meaning |
|---|---|---|
| `moveLeft`, `moveRight` | Held | Move with delayed auto shift and auto repeat (core) |
| `stepLeft`, `stepRight` | One-shot | Move exactly one column, no repeat (used by drag gestures) |
| `softDrop` | Held | Fall at 20 times current gravity |
| `hardDrop` | One-shot | Drop to the ghost position and lock |
| `rotateCW`, `rotateCCW` | One-shot | Rotate with SRS wall kicks |
| `hold` | One-shot | Swap with the hold slot |
| `pause` | App-level | Pause (not sent to the core) |

## Keyboard defaults (all remappable)

Defaults use `KeyboardEvent.code` so they work across layouts.

| Action | Default keys |
|---|---|
| `moveLeft` | `ArrowLeft` |
| `moveRight` | `ArrowRight` |
| `softDrop` | `ArrowDown` |
| `hardDrop` | `Space` |
| `rotateCW` | `ArrowUp`, `KeyX` |
| `rotateCCW` | `KeyZ` |
| `hold` | `KeyC`, `ShiftLeft`, `ShiftRight` |
| `pause` | `Escape`, `KeyP` |

Rules:

- Assigning a key already used by another action swaps the two actions and tells the player.
- Operating-system key repeat (`event.repeat`) is ignored; repeat is the core's job.
- If both horizontal keys are held, the latest press wins; releasing it resumes the other if still held.
- Menus use their own fixed navigation keys (Tab, Shift+Tab, Enter, Space, Escape, arrow keys within lists) and are not remappable.

## Gesture mapping

| Gesture | Action sent |
|---|---|
| Tap on the right half | `rotateCW` |
| Tap on the left half | `rotateCCW` |
| Horizontal drag | `stepLeft` / `stepRight`, one per cell width of travel |
| Slow drag down | `softDrop` press while dragging, release when the finger stops or lifts |
| Fast flick down | `hardDrop` |
| Swipe up | `hold` |
| Corner pause button | `pause` |

Default thresholds (at sensitivity 1.0, scaled 0.5 to 1.5 by the setting) are in [research.md](../research.md) section 5. Only the primary pointer is tracked. The game surface sets `touch-action: none` and `overscroll-behavior: none` and prevents default on `touchmove` there, so no scroll, zoom, pull-to-refresh or back navigation starts during play.

## Auto-pause triggers

The app pauses when the page becomes hidden, the window loses focus, or the page is being hidden (`visibilitychange`, `blur`, `pagehide`). Resuming from any pause runs a 3-second countdown before play continues. Held keys are released on pause and must be pressed again.
