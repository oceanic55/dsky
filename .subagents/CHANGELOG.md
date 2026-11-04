# CHANGELOG — Historical Changes

History only. Current behaviour is documented in `.subagents/overview.md`, `.subagents/workflows.md`, and `AGENTS.md`.

## 2026-11-03 — Native R3 picker, PROG 01 gating, export chain

- PROG 03's bottom-sheet list picker (`#picker-sheet`, `.picker-*` CSS, `.picker-group` / `.picker-item` rows, checkmark, Cancel/backdrop close, `.picker-list` touchmove exemption) is removed. The picker is now a native `<select>` (`#entry-select`) inside `#r3-picker`, overlaid on the unused R3 register row: iOS opens the system wheel (native scrolling, no exemption needed), Chrome/Firefox call `showPicker()`, Safari relies on the tap. It hides via `clearAllFields()` and repopulates after SAVE.
- PROG 01 is gated on a resident file: with no document loaded it opens the PROG 04 new-file prompt first and continues into the R1 sheet only after Create (cancelling the prompt leaves R1 closed). This stops the first entry from being written with COMP REDY dark/green, which happened when R1 was entered without ever running PROG 04.
- OFFL's export gained step 1: `window.showSaveFilePicker` (Chromium) opens a native save dialog with full Finder navigation, before the existing Web Share step and anchor-download fallback. A dismissed dialog, share sheet, or Cancel keeps the document resident.

## 2026-11-03 — Export destination and scrollable picker

- OFFL no longer calls an `<a download>` immediately. It opens an in-app Save / Cancel sheet, then uses `navigator.share({ files })` so iOS users can choose **Save to Files** and a destination. Cancelling either sheet keeps the document resident.
- The anchor download remains as a fallback when file sharing is unavailable. Safari's View / Download prompt and its default destination cannot be customised by a web page.
- PROG 03's picker now scrolls on iOS. The global `touchmove` blocker exempts `.picker-list`, the sheet and list use `touch-action: pan-y`, and the list has `min-height: 0` so the flex child shrinks inside the `70vh` sheet.
- The picker is restyled as an HIG-style inset grouped list: 44 pt minimum rows, hairline separators, and a blue checkmark on the loaded entry.
- Removed the old per-row white cards, the duplicate `pointerup` selection handler, and the non-scrolling picker layout.

## 2026-11-02 — CLR unloads the displayed entry

- CLR now clears the display and detaches the selected PROG 03 entry from view without modifying or deleting its resident log row.
- Entry contents are edited through PROG 01. CLR also discards an unsaved new R1 draft; the resident file and any unsaved constant/trail changes remain intact.

## 2026-11-02 — Entry unload and COMP REDY states

- SAVE now unloads R1/R2 and clears `currentEntry` after writing, while the document stays resident. The next SAVE creates a new entry instead of editing the just-saved one.
- COMP REDY gained an orange state: green = document resident with no entry loaded, orange = an unchanged entry is loaded, red = unsaved new/modified entry or settings, dark = no document.
- Previously green meant only "document resident and saved"; an entry stayed loaded after SAVE and was silently edited by the next save.
- Unsaved drafts are discarded without touching the document when the user loads another entry, copies with VERB/NOUN, opens another PROG, or offloads with OFFL.
- SAVE with no document, no R1, and default settings is a no-op.

## 2026-11-02 — Resident-document redesign

Replaced the append-and-download model with an in-memory resident document.

**Keys**

- `RSET` became `SAVE`; it writes to the resident document and never downloads.
- `ENTR` became `LOAD` / `OFFL`; OFFL downloads the resident document and unloads it.
- PROG gained mode `04`.

**PROG editors**

- PROG 01, 02, 03, and 04 open their editors directly; the old "select PROG, then press RSET" gate was removed.
- PROG 01 split R1 into two fields, R1 A and R1 B, joined by one space.
- PROG 02 edits the constant and trail together.
- PROG 03 lists only full R1 values, alphabetically; the timestamp/meta line was removed.
- PROG 04 creates a blank document under a prompted file name.

**Data**

- JSON reduced to `{ log: [{ timestamp, r1 }], constant, trail }`.
- Removed per-entry `prog`, `verb`, `noun`, `r2`, `r3`, `display`, and `modified` fields.
- Removed all `localStorage` use (`dsky_log` key no longer read or written).
- Logs are sorted by full R1, case-insensitive, on load and save.
- R1 limit raised from 10 to 20 to 30 characters; only the first 5 render.
- R2 is the first 5 characters of R1 B; the old R2-12 / R2-45 derivations are gone.

**Copy keys**

- VERB copies R1 B, or `*empty*` when R1 B has no content.
- NOUN copies `R1[0]` uppercase + `R1[1]` lowercase + constant + trail.

**Selection and deletion**

- Selecting an entry in PROG 03 and then pressing CLR + SAVE deletes it.
- Switching entries discards unsaved edits; COMP REDY returns to green.
- COMP REDY reflects saved/dirty/no-file state (green/red/dark); it previously just meant "file loaded".

**Removed UI**

- Analysis sidebar, scale slider, "Hold pressed" toggle, and sound toggle.
- R2 copy icon and its `.r2-copy` styles.
- RSET key red active-mode styling.

**Added UI**

- Two decorative screws on the panel top, aligned with the keypad screws.
- `--s` is now a fit-to-width formula capped at 1.0; the old fixed default was 1.2×.

## 2026-11-01 — Earlier single-file iteration (removed)

- ENTR downloaded `dsky.json` on every press, cleared the display, and returned the key to LOAD.
- Entries were appended to `localStorage["dsky_log"]` and stored PROG/VERB/NOUN/R1/R2/R3 plus a `display` summary and a `modified` flag.
- CLR "slated" a loaded entry for in-place update or deletion on the next ENTR save.
- R1 was capped at 10 characters with a 5-character display.
- Constant editing lived behind `PROG 02` + RSET, which turned the key red.
- Earlier artifacts were moved to `development/` (`1.html`, `6.html`, `7.html`, `8.html`, `9.html`, `10.html`). Reference images moved to `Documents/`.
- An `extra/` CSS/JS split was started and abandoned; the artifact remained single-file.


## 2026-11-02 — Message box (`.caption`)

- The static `.caption` label (`PROG VN R1 R2 R3 · N01 21:07`) was replaced with a dynamic message box driven by `messageState` and `updateMessage()`.
- Nine status/error states: NO_FILE, FILE_LOADED, ENTRY_SAVED, SAVE_ERROR, R1_NOT_SAVED, ENTRY_CLEAR, ENTRY_LOADED, R1B_COPY, KY_COPY.
- Timed messages auto-revert: ENTRY_SAVED and SAVE_ERROR after 3 s; ENTRY_CLEAR, R1B_COPY, and KY_COPY after 5 s.
- VERB/NOUN copy confirmations revert to the full R1 B content when an entry is loaded.
- CSS `data-state` attribute recolours the text: mint for success, orange for warning, red for error, grey for info.
- `saveCurrent()` wrapped in try/catch to emit SAVE_ERROR on failure.
- See `.subagents/messages.md` for full detail.
