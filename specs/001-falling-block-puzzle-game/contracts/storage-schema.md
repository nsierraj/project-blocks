# Contract: Saved Data Schema

All saved data is local to the browser. Keys are prefixed because GitHub Pages serves every project site of an account from one origin, so unprefixed keys can collide with other projects. No personal data is stored (FR-047).

## Keys

| Key | Content |
|---|---|
| `blocks:meta` | `{ "schemaVersion": 1 }` |
| `blocks:settings` | Settings object (below) |
| `blocks:scores` | Scores object (below) |

The prefix `blocks` is the app id (provisional name; see plan). Changing it is a data migration.

## Settings object

```json
{
  "keyMap": { "moveLeft": ["ArrowLeft"], "moveRight": ["ArrowRight"], "softDrop": ["ArrowDown"],
              "hardDrop": ["Space"], "rotateCW": ["ArrowUp", "KeyX"], "rotateCCW": ["KeyZ"],
              "hold": ["KeyC", "ShiftLeft", "ShiftRight"], "pause": ["Escape", "KeyP"] },
  "dasMs": 167,
  "arrMs": 33,
  "gestureSensitivity": 1.0,
  "theme": "system",
  "soundOn": true,
  "volume": 0.7,
  "hapticsOn": true,
  "gestureOverlaySeen": false,
  "marathonStartLevel": 1,
  "marathonEndless": false
}
```

Validation on read: `dasMs` 50 to 300, `arrMs` 0 to 100, `gestureSensitivity` 0.5 to 1.5, `volume` 0 to 1, `theme` one of `system`, `light`, `dark`, `marathonStartLevel` integer 1 to 15. Any invalid field is replaced by its default; other valid fields are kept. A key code may appear for only one action.

## Scores object

```json
{
  "marathon-150":     [ { "score": 0, "timeMs": 0, "lines": 0, "level": 1, "date": "2026-10-26T12:00:00Z", "seed": 0 } ],
  "marathon-endless": [],
  "sprint":           [],
  "ultra":            [],
  "zen":              []
}
```

Each list holds at most 10 entries, kept sorted by its ranking key (score or lines descending, time ascending for `sprint`) then by earlier `date`. Sprint entries are stored only for completed games (40 lines).

## Versioning and migration

- `schemaVersion` starts at 1. When the app finds an older version it runs migration functions in order and writes the result; when it finds a newer version than it knows, it leaves the data untouched, runs with defaults in memory, and does not overwrite.
- Corrupt JSON in a key replaces only that key with defaults; the other keys are kept.
- Updating the app (new service worker) must never clear these keys. Only the player's "Reset scores and settings" action, after confirmation, removes keys that start with `blocks:`.

## Unavailable or full storage

If reading or writing throws (blocked storage, private mode, quota), the wrapper switches to an in-memory store for that session: settings work until the page closes, scores are not saved, and no blocking error is shown (FR-037).
