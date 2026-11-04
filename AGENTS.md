# AGENTS.md — DSKY Project Guide

## Start Here

DSKY is a standalone HTML/CSS/JavaScript analysis artifact reproducing the Apollo Guidance Computer DSKY interface. It is **not** an Xcode project. The current artifact (`index.html`) renders the full display panel plus a 3×2 function keypad, with bottom sheets for the PROG/SAVE/LOAD workflow. All geometry and colors are derived from a Swift/SwiftUI source (`ContentView.swift / DisplayPanelView / KeyView / SevenSegmentView`).

## Working Rules

- Progress visibility: `Progress: analyzing codebase…`, `Progress: generating patch…`, `Progress: running verification…`
- Never silent >10s during internal work.
- Explain: non-engineer summary first, then user-visible consequences grouped by severity, then implementation details.
- **Vision gate (0):** attached image not visible to model → stop and notify. Never infer.

## Build and Verify

- Requirements: any modern browser (Safari, Chrome, Firefox). No build step.
- Entry point: open `index.html` directly in a browser (`file://` works; Audio will play after a user gesture).
- Assets loaded at runtime: `img/screw.png`, `sd/4.wav`, `sd/5.wav`, `sd/7.wav` (`4.wav` is loaded but unused).
- No automated tests. Verify visually: display panel, keypad, PROG 01–04 sheets, SAVE, OFFL/LOAD, COMP REDY colours.
- File tree: `index.html` (current), `development/` (earlier artifacts), `Documents/` (reference images). See `.subagents/overview.md` for full structure.

## Critical Constraints

0. **Vision gate** — image not visible to model → stop, notify. Never infer.
1. All geometry (sizes, positions, radii) must match the Swift source values documented in `index.html`'s header comment and in `.subagents/overview.md`. Do not approximate.
2. Segment polygon vertices are exact normalized coordinates from `SevenSegmentView`. Never replace them with a font.
3. Mint accent color is `#35d39a` (DSKY.Display token). Do not use any other value.
4. Sound uses `<Audio>` elements, not Web Audio API, so `file://` loads work. Do not switch to `fetch()` or `AudioContext` for the click sounds.
5. Three voices per clip (`VOICES = 3`) allow rapid overlapping presses. Do not reduce to a single `<audio>` element.
6. Press animation: `scale(0.93)`, 70 ms `ease-out`, 90 ms flash (`PRESS_MS`). Preserve both durations.
7. Scale is a CSS custom property `--s` on `:root`; the `.stage` uses `transform: scale(var(--s))`. Do not restructure this.
8. `index.html` is self-contained (single file, no bundler). Keep it that way unless explicitly asked to split.
9. This is an analysis artifact — do not treat it as production code or add a backend, auth, or data persistence unless asked.
10. `Documents/` images and `development/` artifacts are reference material only. Never delete them.

## Runtime Rules (see `.subagents/workflows.md`)

- Button `data-sound` values map to clip files: `"4"` → `sd/4.wav`, `"5"` → `sd/5.wav`, `"7"` → `sd/7.wav`.
- Current mapping: `sd/5.wav` plays for VERB, CLR, NOUN, PROG, SAVE; `sd/7.wav` plays for LOAD/OFFL. `sd/4.wav` is loaded but unused.
- `VERB` copies R1 B (everything after the first space in R1); when R1 B is empty it copies the literal text `*empty*`. `NOUN` copies `R1[0]` uppercase + `R1[1]` lowercase + constant + trail.
- `PROG` 01 opens the R1 entry sheet — but with no file resident it first runs the PROG 04 new-file prompt, then continues into R1 (cancelling the prompt leaves R1 closed); 02 opens the constant/trail sheet, 03 reveals the R3 entry picker, and 04 opens the new-file prompt — no SAVE press required.
- The PROG 03 picker is a native `<select>` overlaid on the unused R3 register area (`#r3-picker`). iOS opens the system wheel on tap (native momentum scrolling); Chrome/Firefox auto-open via `showPicker()`. Keep the global `touchmove` blocker as-is — it does not affect native controls; never rebuild a custom scrolling list.
- `PROG 04` creates a blank resident log (no entries, default constant, empty trail) under a user-supplied file name.
- `SAVE` writes the displayed entry/constant/trail into the resident log, sorts the log alphabetically by R1, unloads R1 (the document stays resident), and stays in-app (no download prompt).
- `CLR` clears the display and unloads a selected entry from view without changing the resident log; use PROG 01 to edit entry contents.
- `OFFL` opens an in-app sheet with SAVE, OFFLOAD, and CANCEL. SAVE exports through the first available channel (`showSaveFilePicker`, share sheet, or download fallback) and unloads only after success; OFFLOAD discards the resident document and unloads it without exporting; CANCEL keeps it resident. Neither action saves edited entries.
- COMP REDY is green while a file is resident with no entry loaded and no unsaved changes, orange while an unchanged entry is loaded, red while a new/modified entry or setting is unsaved, and dark when no file is loaded.
- R1 is entered as two fields in the PROG 01 sheet — **R1 A** and **R1 B** — joined by a single space into a 30-character max R1 value; only the first 5 characters are rendered.
- R2 displays the first 5 characters of R1 B.
- JSON shape: `{ "log": [{ "timestamp", "r1" }], "constant": "12 chars", "trail": "2 chars" }`.
- Pointer-down fires the sound; pointer-up fires the activation (press flash) and the key action.
- Keyboard activation fires on `click` with `event.detail === 0`; the pointer path does not re-sound.
- The `.caption` element is a message box with 9 status/error states; see `.subagents/messages.md`. Timed states auto-revert (3 s for SAVE results, 5 s for CLR and copy confirmations).

## Quick Reference

| Setting | Value |
|---|---|
| Current artifact | `index.html` |
| Stage size | 274 × 561 pt |
| Bezel | 201 × 341, radius 9 |
| Screen | 189 × 329, radius 5 |
| Digit cell | 26 × 30 pt |
| Register cell | 26 × 30 pt (sign 12.1 × 30) |
| Key grid | 3 × 2, 68 pt cells, 7 pt row / 9 pt col gap |
| Screws | 22 pt: 4 keypad corners + 2 panel top |
| Mint token | `#35d39a` |
| Press scale | 0.93, 70 ms ease-out, 90 ms flash |
| Sound voices | 3 per clip (4.wav / 5.wav / 7.wav) |
| R1 limit | 30 chars stored, 5 rendered |
| R3 picker | Native `<select>` overlay, revealed by PROG 03 |
| Message box | `.caption`, 9 states, CSS `data-state` colour-coded |
| Default scale | Fit-to-width, capped at 1.0× |

## Documentation Rules

- Every source change updates `AGENTS.md` + affected `.subagents/` docs in the same commit.
- **AGENTS.md ≤ 150 lines** (excl. reference table). If exceeded, move detail to `.subagents/` and replace with pointer.
- Docs describe verified present implementation only. Removed behavior → `.subagents/CHANGELOG.md`; runtime detail → `.subagents/workflows.md`.

## Detailed Reference

| File | Use for |
|---|---|
| `.subagents/overview.md` | Architecture, file structure, component geometry, color tokens |
| `.subagents/workflows.md` | Runtime state, key behaviour, sheets, JSON contract |
| `.subagents/messages.md` | Message box states, triggers, timing, integration points |
| `.subagents/documentation.md` | Documentation organization and enforcement rules |
| `.subagents/CHANGELOG.md` | Historical changes only |
