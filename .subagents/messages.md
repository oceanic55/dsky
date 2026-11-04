# messages.md — Message Box (`.caption`)

The `.caption` element sits below the stage and displays status and error messages. It was previously a static label (`PROG VN R1 R2 R3 · N01 21:07`); it is now a dynamic message box driven by `messageState` and `updateMessage()`.

Current behaviour only. Historical changes live in `.subagents/CHANGELOG.md`.

## Element and CSS

The existing `.caption` class is reused. Key CSS properties:

| Property | Value | Purpose |
|---|---|---|
| `font-size` | `calc(11px * var(--s))` | Scales with the stage |
| `color` | `#77777b` (default / info) | Neutral grey |
| `max-width` | `calc(274px * var(--s))` | Matches panel width |
| `overflow` / `text-overflow` | `hidden` / `ellipsis` | Long content truncates with `…` |
| `white-space` | `nowrap` | Single line |
| `transition` | `color 0.3s ease, opacity 0.3s ease` | Smooth colour changes |

### Colour-coded states (`data-state` attribute)

`updateMessage()` sets `caption.dataset.state` so CSS can recolour the text:

| `data-state` | Colour | States |
|---|---|---|
| `info` (default) | `#77777b` grey | NO_FILE, FILE_LOADED, ENTRY_CLEAR, ENTRY_LOADED, R1_NOT_SAVED |
| `success` | `#35d39a` mint | ENTRY_SAVED, R1B_COPY, KY_COPY |
| `warning` | `#ffa726` orange | R1_NOT_SAVED |
| `error` | `#ff6b6b` red | SAVE_ERROR |

## State Model

```javascript
const messageState = {
  current: 'NO_FILE',   // active state key
  previous: null,       // last non-timed state (for revert)
  timeoutId: null,      // active setTimeout handle
  STATES: { … }         // key → display text (with {fileName} / {r1bContent} placeholders)
};
```

- `current` — the key of the state currently displayed.
- `previous` — the last persistent state before a timed message was shown. Timed messages do not overwrite `previous`, so the revert target is preserved.
- `timeoutId` — handle for the pending auto-revert timer. Cleared on every `updateMessage()` call so rapid state changes cancel the previous timer.

## The Nine States

| # | Key | Trigger | Display text | Timed? | Revert target |
|---|---|---|---|---|---|
| 1 | `NO_FILE` | No file loaded or user pressed OFFL | `AGC DSKY Mini - No File` | No | — |
| 2 | `FILE_LOADED` | A JSON file is loaded or a timed message expires with a resident file | `AGC DSKY Mini - {fileName}` | No | — |
| 3 | `ENTRY_SAVED` | SAVE succeeded | `ENTRY SAVED` | 3 s | FILE_LOADED or NO_FILE |
| 4 | `SAVE_ERROR` | SAVE threw an exception | `SAVE ERROR` | 3 s | FILE_LOADED or NO_FILE |
| 5 | `R1_NOT_SAVED` | Entry loaded and `dirty === true` (set in `refreshDirty`) | `R1 NOT SAVED` | No | — |
| 6 | `ENTRY_CLEAR` | User pressed CLR | `ENTRY CLEAR` | 5 s | FILE_LOADED or NO_FILE |
| 7 | `ENTRY_LOADED` | Entry selected via PROG 03 picker | Full R1 B contents | No | — |
| 8 | `R1B_COPY` | User pressed VERB (copies R1 B) | `R1 B COPY` | 5 s | ENTRY_LOADED (R1 B) or previous |
| 9 | `KY_COPY` | User pressed NOUN (copies key string) | `KY COPY` | 5 s | ENTRY_LOADED (R1 B) or previous |

### State 7 — ENTRY_LOADED

Displays the **full R1 B content** (everything after the first space in R1). The CSS `text-overflow: ellipsis` truncates long content visually without changing the stored text.

R1 B extraction: `r1.split(' ').slice(1).join(' ')`. When R1 has no space, R1 B is empty and the message is blank.

### Timed-message revert logic

For states 8 and 9 (R1B_COPY, KY_COPY):
- After 5 s, if `currentEntry !== null`, revert to `ENTRY_LOADED` with the current entry's R1 B.
- After 5 s, if no entry is loaded, revert to `messageState.previous`.

For states 3, 4, 6 (ENTRY_SAVED, SAVE_ERROR, ENTRY_CLEAR):
- After the timeout, if `loadedLog !== null`, revert to `FILE_LOADED` with the current `fileName`.
- After the timeout, if no file is resident, revert to `NO_FILE`.

## `updateMessage(state, data)` — API

```
updateMessage(stateKey, data?)
```

- `stateKey` — one of the nine keys above.
- `data` — optional object:
  - `{ fileName }` for `FILE_LOADED`.
  - `{ r1bContent }` for `ENTRY_LOADED`.

Behaviour:
1. Clears any pending timeout (`messageState.timeoutId`).
2. If the new state is not a timed message, stores the previous state in `messageState.previous`.
3. Sets `messageState.current` to the new key.
4. Resolves the display text (template substitution for `FILE_LOADED`, raw content for `ENTRY_LOADED`).
5. Updates `caption.textContent` and `caption.dataset.state`.
6. If the state is timed, schedules the auto-revert via `setTimeout`.

## Integration Points

| Function | Trigger | Message set |
|---|---|---|
| Initialisation (after `syncCompRedy`) | Page load | `NO_FILE` |
| `loadEntry()` success | File loaded | `FILE_LOADED` with `fileName` |
| `unloadFile()` | OFFL pressed | `NO_FILE` |
| `selectEntry()` | PROG 03 entry picked | `ENTRY_LOADED` with R1 B |
| `refreshDirty()` | Entry modified while loaded | `R1_NOT_SAVED` (when `currentEntry && dirty`) |
| `saveCurrent()` success | SAVE pressed | `ENTRY_SAVED` |
| `saveCurrent()` catch | SAVE failed | `SAVE_ERROR` |
| `handleKey('CLR')` | CLR pressed | `ENTRY_CLEAR` |
| `handleKey('VERB')` | VERB pressed | `R1B_COPY` |
| `handleKey('NOUN')` | NOUN pressed | `KY_COPY` |

## Invariants

- Only one timeout is active at a time; each `updateMessage()` call clears the previous timer.
- `messageState.previous` is never updated by a timed message, so the revert target is always the last persistent state.
- The `.caption` element is the sole DOM node updated; no new elements are created.
- The CSS `data-state` attribute is always set (defaults to `info`).
