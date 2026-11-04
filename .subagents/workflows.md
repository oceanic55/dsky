# workflows.md — Runtime Behaviour and Data Contract

Current behaviour only. Removed behaviour lives in `.subagents/CHANGELOG.md`.

## Key Map

| Key | `data-key` | Action |
|---|---|---|
| VERB | `VERB` | Copy R1 B to the clipboard |
| CLR | `CLR` | Unload the displayed entry and clear the display |
| LOAD / OFFL | `ENTR` | Toggle file load / offload |
| NOUN | `NOUN` | Copy the composed NOUN string |
| PROG | `PROG` | Open the PROG numpad |
| SAVE | `SAVE` | Write the displayed entry and settings into the resident document |

The key's visible label changes with state: `LOAD` when no file is resident, `OFFL` when one is.

## State Model

| Variable | Type | Meaning |
|---|---|---|
| `displayState` | object | `{ prog, verb, noun, r1, r2, r3 }`; every renderer writes here |
| `loadedLog` | object \| null | Resident `{ log, constant, trail }` |
| `currentEntry` | object \| null | Entry selected in PROG 03; cleared after SAVE and on offload |
| `dirty` | boolean | Resident document has unsaved differences |
| `constant` | string | 12-character constant (default `779900009999`) |
| `trail` | string | 2-character trail (default empty) |
| `fileName` | string | Export name, default `dsky.json` |
| `messageState` | object | `{ current, previous, timeoutId, STATES }`; drives the `.caption` message box — see `.subagents/messages.md` |

`displayState.r1` stores the **full** R1 value (up to 30 characters). Only the first 5 render in the R1 register. `displayState.r2` stores exactly what R2 shows.

## PROG Flows

PROG accepts a single digit `1`–`4`; the display is padded to `01`–`04`. Selecting a PROG opens its editor after the numpad closes (220 ms delay); no SAVE press is required. PROG 01 without a resident document runs the PROG 04 prompt first and only continues into R1 after Create.

| PROG | Opens | Saves into |
|---|---|---|
| `01` | R1 sheet — R1 A and R1 B fields (PROG 04 prompt first when no file is resident) | `displayState.r1` |
| `02` | Constant / trail modal | `constant`, `trail` |
| `03` | Entry picker (native `<select>` in R3) | `currentEntry`, `displayState.r1` |
| `04` | New-file prompt | New blank resident document |

### PROG 01 — R1 entry

- **Gating:** when no document is resident, PROG 01 opens the PROG 04 new-file prompt first; the R1 sheet opens only after Create. Backdrop / Escape on the prompt cancels both, so R1 can never be written without a resident file (which kept COMP REDY dark on the first entry).
- Two fields: **R1 A** and **R1 B**.
- The stored value is `R1 A + " " + R1 B` when both are present; if only R1 A is present, that is the value; R1 B alone becomes the whole R1.
- Combined length is capped at 30 characters, including the separating space. R1 A may use up to 30; R1 B gets the remaining room.
- Return in R1 A moves focus to R1 B. Return in R1 B, or Done, commits.
- Opening prefills by splitting the current R1 at its first space.
- R1 renders the first 5 characters in uppercase glyphs. R2 renders the first 5 characters of R1 B.
- Committing re-evaluates `dirty`.

### PROG 02 — Constant and trail

- Two fields: 12-character constant and 2-character trail.
- The constant must be exactly 12 characters to save; the trail may be empty.
- Saving updates `constant` / `trail`, re-renders R2, and re-evaluates `dirty`.
- Escape or backdrop cancels without saving.

### PROG 03 — Entry picker

- A native `<select>` (`#entry-select`) inside `#r3-picker`, absolutely overlaid on the unused R3 register row (`.reg` is its positioning context). Hidden by default; PROG 03 reveals it and pulses the mint outline.
- iOS opens the system wheel picker on tap — native momentum scrolling, no page-scroll exemption needed. Chrome/Firefox call `showPicker()` automatically when transient activation allows; Safari has no `showPicker()`, so the pulse invites the tap.
- Options: placeholder `NO FILE` / `NO ENTRIES` / `SELECT ENTRY…`, then one option per entry (full R1, alphabetical by `compareEntries`), value = `timestamp`.
- Selecting loads the entry: clears the display, renders that entry's R1 (and therefore R2), sets `currentEntry`, re-reveals the picker with that selection, and turns COMP REDY orange. Unsaved edits to the previously displayed entry are discarded; the document itself is unchanged, and it stays resident.
- The picker hides again inside `clearAllFields()` (CLR, LOAD, OFFL, new file, entry load). SAVE repopulates it in place when visible, because SAVE regenerates timestamps.
- The global `touchmove` blocker has no exemption; native select wheels and OS dialogs are unaffected by it.

### PROG 04 — New file

- Prompts for a file name, default `dsky`. `.json` is appended if missing.
- Creates `{ log: [], constant: "779900009999", trail: "" }`, sets it resident, and lights COMP REDY green.
- Any unsaved changes in the previously resident document are discarded without a warning.
- When chained from PROG 01 (no file resident), Create continues into the R1 sheet after the modal slides out (340 ms); cancelling the prompt does not open R1.

## SAVE

SAVE never downloads a file and never unloads the document. It unloads only the entry from R1, so the next SAVE creates a new entry instead of editing the previous one.

1. Starts from the resident `loadedLog.log` (empty when no document is resident; SAVE then creates a new in-memory document using the current `constant` and `trail`).
2. Updates the loaded entry by timestamp, or appends a new `{ timestamp, r1 }`.
3. If `currentEntry` was selected and R1 is now empty, that entry is **deleted** (CLR itself only unloads it from view).
4. Trims R1, slices to 30 characters, and ignores an empty new entry.
5. Sorts the log alphabetically by full R1 (`localeCompare`, case-insensitive).
6. Writes `{ log, constant, trail }` back to `loadedLog`, then unloads R1 and R2, sets `currentEntry` to `null`, clears `dirty`, and returns COMP REDY to green.
7. With no document resident, no R1 value, and default constant/trail, SAVE is a no-op.

CLR does not delete entries; it only unloads the displayed entry from view.

## CLR

Clears PROG, VERB, NOUN, R1, R2, R3 and the display, hides the R3 entry picker, and sets `currentEntry` to `null`. This unloads a selected entry from view without changing or deleting its row in the resident log. Use PROG 01 to edit displayed R1 contents. CLR does not unload the document or touch `constant` / `trail`; it re-evaluates `dirty`, so any unsaved settings remain marked red. Clearing a new unsaved R1 draft discards that draft.

## OFFL / LOAD

The key is `data-key="ENTR"`; the label reflects state.

| State | Press behaviour |
|---|---|
| Document resident (`loadedLog !== null`) | **OFFL** — open the in-app SAVE / OFFLOAD / CANCEL sheet |
| No document resident | **LOAD** — open the file picker |

OFFL never goes straight to Safari's download prompt. The three-button sheet (`#export-sheet`) offers:

1. **CANCEL**, or a backdrop tap, closes the sheet and keeps the document resident.
2. **OFFLOAD** discards the resident document without exporting it, then unloads the document and clears the display.
3. **SAVE** runs the export chain, first channel that applies wins, and the file unloads only after a successful save:
   1. `window.showSaveFilePicker` (Chromium/Edge only) — native save dialog with full Finder navigation (folder + file name). Cancelling it (`AbortError`) keeps the document resident.
   2. `navigator.share({ files })` when `canShare({ files })` allows it — the iOS share sheet with **Save to Files**. Cancelling it (`AbortError`) keeps the document resident.
   3. `<a download>` anchor fallback — on iOS that shows Safari's own View / Download prompt, whose wording and destination cannot be changed by the page.
4. The save dialog must be requested inside the click's user activation (before any `await`), so `showSaveFilePicker` runs first.

Offload resets `loadedLog`, `currentEntry`, `dirty`, `constant`, `trail`, and `fileName`, clears the display, and darkens COMP REDY.

### Loading a file

1. `FileReader.readAsText` (no `fetch`), so `file://` works.
2. Accepts `{ log: [...] }` or a bare single-entry object.
3. Each row becomes `{ timestamp, r1 }`; rows with an empty R1 are dropped; R1 is sliced to 30 characters.
4. `constant` is used only when it is exactly 12 characters; otherwise the current in-memory constant is kept. `trail` is sliced to 2.
5. The log is sorted alphabetically and `fileName` becomes the loaded file's name.
6. The display clears and COMP REDY turns green.

## COMP REDY

| Colour | Meaning |
|---|---|
| Green | A document is resident, no entry is loaded, and nothing is unsaved |
| Orange | An entry is loaded and unchanged (viewing only) |
| Red | A new entry, a modified loaded entry, or changed constant/trail is unsaved |
| Dark | No document is resident |

`refreshDirty()` compares `displayState.r1` with `currentEntry.r1`, or treats any non-empty R1 as unsaved when no entry is loaded, and compares `constant` / `trail` with the resident document. With no document resident, everything is considered clean and the light is dark.

## Copy Keys

| Key | Output |
|---|---|
| VERB | R1 B — the text after the first space in R1. When R1 B is empty, the literal `*empty*` |
| NOUN | `R1[0]` uppercase + `R1[1]` lowercase + constant + trail, concatenated with no separators |

Example: R1 `entry01 retr@76-see`, constant `000000000000`, trail `##` → NOUN copies `En000000000000##`.

Clipboard uses `navigator.clipboard.writeText` with a `document.execCommand('copy')` textarea fallback.

After VERB or NOUN copies, the `.caption` message box shows `R1 B COPY` or `KY COPY` for 5 s, then reverts to the full R1 B content of the loaded entry (or the previous state when no entry is loaded).

## Message Box

The `.caption` element below the stage displays status and error messages through 9 states managed by `messageState` and `updateMessage()`. Full detail is in `.subagents/messages.md`.

| State | Trigger | Timed | Revert |
|---|---|---|---|
| `NO_FILE` | No file loaded / OFFL | — | — |
| `FILE_LOADED` | File loaded | — | — |
| `ENTRY_SAVED` | SAVE succeeded | 3 s | → FILE_LOADED / NO_FILE |
| `SAVE_ERROR` | SAVE threw | 3 s | → FILE_LOADED / NO_FILE |
| `R1_NOT_SAVED` | Entry loaded + dirty | — | — |
| `ENTRY_CLEAR` | CLR pressed | 5 s | → FILE_LOADED / NO_FILE |
| `ENTRY_LOADED` | Entry selected | — | — |
| `R1B_COPY` | VERB pressed | 5 s | → R1 B content / previous |
| `KY_COPY` | NOUN pressed | 5 s | → R1 B content / previous |

The CSS `data-state` attribute recolours the text: mint `#35d39a` for success (saved, copies), orange `#ffa726` for warning (not saved), red `#ff6b6b` for error, grey `#77777b` for info.

## JSON Contract

```json
{
  "log": [
    {
      "timestamp": "2026-11-02T08:35:19.506Z",
      "r1": "entry01 retr@76-see"
    }
  ],
  "constant": "000000000000",
  "trail": "##"
}
```

- `log` — entries sorted by full R1, case-insensitive.
- `timestamp` — ISO 8601, regenerated when an entry is updated.
- `r1` — full R1, up to 30 characters.
- `constant` — exactly 12 characters.
- `trail` — up to 2 characters.

Older rows containing `prog`, `verb`, `noun`, `r2`, `r3`, `display`, or `modified` are read for `timestamp` and `r1` only; those extra keys are dropped on the next SAVE.

## Sheets and Limits

| Surface | Element | Close paths |
|---|---|---|
| PROG numpad | `#numpad-sheet` | Done, backdrop |
| R1 entry | `#alpha-sheet` | Done, backdrop |
| Export | `#export-sheet` | SAVE, OFFLOAD, CANCEL, backdrop |
| Constant / trail | dynamic modal | Done / Return, Escape, backdrop |
| New file | dynamic modal | Create / Return, Escape, backdrop |

The PROG 03 entry picker (`#r3-picker`) is not a sheet: it is a native `<select>` in the R3 register row, so it has no close path — it hides via `clearAllFields()`. Only one sheet is open at a time; opening a PROG editor waits 220 ms for the numpad close animation.

## Invariants

- `displayState.r1` is the full stored value; the R1 register renders only 5 characters.
- `displayState.r2` is derived purely from R1 B; it is never entered manually.
- SAVE is in-memory only; OFFL is the only export path.
- SAVE unloads the entry from R1 while the document stays resident.
- OFFL offers SAVE (export then unload), OFFLOAD (discard then unload), or CANCEL (keep resident); dismissed native save/share dialogs also keep it resident.
- R3 renders blank; its only overlay is the PROG 03 entry picker (`#r3-picker`), which never writes `displayState.r3`.
