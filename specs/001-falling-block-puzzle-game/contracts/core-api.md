# Contract: Game Core API

The only interface between the pure rules (`src/core`) and everything else (renderer, input, audio, haptics, storage, UI). The core has no DOM, audio, storage or timer dependency (Constitution II). Signatures are shown as TypeScript-style interfaces to fix the shape; they are a contract, not implementation.

## Creating a game

```ts
createGame(options: {
  mode: 'marathon' | 'sprint' | 'ultra' | 'zen';
  seed: number;                 // 32-bit integer, supplied by the caller
  startLevel?: number;          // Marathon only, 1 to 15, default 1
  endless?: boolean;            // Marathon only, default false
  settings: { dasMs: number; arrMs: number };
}): Game
```

## Advancing the game

```ts
interface Game {
  // Apply input events (sorted by t) then advance the clock to targetMs, in 1 ms ticks.
  advance(targetMs: number, inputs: InputEvent[]): void;

  // Read-only view for rendering. Does not allocate on the hot path beyond reuse.
  snapshot(): Snapshot;

  // Drain what happened since the last call (for sound, haptics, callouts).
  drainEvents(): GameEvent[];

  // End the game by player choice (Zen's only end).
  quit(): void;
}
```

Rules:

- `targetMs` is simulation time, never wall-clock. The app converts real elapsed time to whole milliseconds and does not call `advance` while paused, backgrounded or counting down.
- `t` of each input is a simulation millisecond between the last `targetMs` and the new one. The core applies each input at its `t`.
- Identical `options`, identical input sequences and identical `targetMs` sequence MUST give identical snapshots and events, independent of how the calls are split across frames (replay tests, SC-006, SC-010).

## Input events

```ts
type InputEvent = {
  t: number;
  type: 'press' | 'release';
  action: Action;     // see input-actions.md
};
```

Held actions (`moveLeft`, `moveRight`, `softDrop`) use press and release; DAS and ARR run inside the core. One-shot actions (`rotateCW`, `rotateCCW`, `hardDrop`, `hold`, `stepLeft`, `stepRight`) use a press only; a release is ignored.

## Snapshot

```ts
interface Snapshot {
  status: 'playing' | 'ended';
  endReason?: 'goal' | 'time' | 'top-out' | 'quit';
  mode: Mode;
  elapsedMs: number;
  remainingMs?: number;           // Ultra
  level: number;
  score: number;
  lines: number;
  combo: number;
  backToBack: boolean;
  cells: ReadonlyArray<ReadonlyArray<PieceType | null>>; // 40 rows x 10 columns, row 0 at the bottom
  active: { type: PieceType; rotation: Rotation; cells: Cell[] } | null;
  ghost: { cells: Cell[] } | null;
  hold: { type: PieceType | null; used: boolean };
  next: PieceType[];              // 5 entries
  lockProgress: number | null;    // 0 to 1, for optional lock-delay feedback
  seed: number;
}
```

## Events

```ts
type GameEvent =
  | { type: 'move' | 'rotate' | 'hold' | 'lock' }
  | { type: 'hardDrop'; cells: number }
  | { type: 'lineClear'; lines: 1 | 2 | 3 | 4; tspin: 'none' | 'mini' | 'full'; combo: number; backToBack: boolean; points: number }
  | { type: 'tspin'; kind: 'mini' | 'full'; lines: number }
  | { type: 'levelUp'; level: number }
  | { type: 'boardCleared' }                      // Zen top-out reset
  | { type: 'ended'; reason: 'goal' | 'time' | 'top-out' | 'quit' };
```

## Pure helpers (also exported, for tests and the renderer)

- `gravityMsPerRow(level: number): number` — the table in research.md 4.2.
- `kicks(piece: PieceType, from: Rotation, to: Rotation): Offset[]` — SRS tables.
- `scoreClear(args): { points: number; ... }` — pure scoring from level, line count, T-spin kind and back-to-back/combo state.
- `detectTSpin(board, piece, lastAction, lastKickIndex): 'none' | 'mini' | 'full'`.

## Compatibility

Changing event shapes or the determinism guarantee is a breaking change to this contract and requires updating the replay tests in the same change.
