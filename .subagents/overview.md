# overview.md — DSKY Project Architecture

## Project Orientation

DSKY is a standalone HTML/CSS/JavaScript analysis artifact reproducing the Apollo Guidance Computer DSKY interface. It is **not** an Xcode project and has no backend, bundler, or build step. All geometry, proportions, colors, and segment shapes derive from a Swift/SwiftUI source (`ContentView.swift / DisplayPanelView / KeyView / SevenSegmentView`).

`index.html` contains the full artifact: display panel, 3×2 keypad, and the interactive sheets used by the PROG/SAVE/LOAD workflow. Runtime behavior is documented in `.subagents/workflows.md`.

## Build and Run

- Open `index.html` directly in any modern browser. `file://` works.
- Audio plays after the first user gesture (browser autoplay policy).
- Required assets are relative: `img/screw.png` and the clips in `sd/`.
- No install, no server, no test runner. Verify visually.

## File Structure

```
DSKY/
├── index.html                  ← The artifact (HTML + CSS + JS in one file)
├── img/
│   └── screw.png               ← Screw image (22 pt)
├── sd/
│   ├── 4.wav                   ← Loaded but unused by any key
│   ├── 5.wav                   ← VERB, CLR, NOUN, PROG, SAVE
│   └── 7.wav                   ← LOAD/OFFL
├── Documents/                  ← Reference images (iosDSKY.jpg, screenshots, SVG)
├── development/                ← Earlier single-file studies (1, 6, 7, 8, 9, 10.html)
├── .subagents/
│   ├── overview.md             ← This file
│   ├── workflows.md            ← Runtime behavior, state model, JSON contract
│   ├── CHANGELOG.md            ← Historical changes only
│   └── documentation.md        ← Documentation organization and enforcement rules
└── AGENTS.md                   ← Launch-critical rules and quick reference
```

## Architecture

`index.html` has three logical sections in one file.

### 1. CSS (in `<style>`)

- `:root { --s: min(1.0, calc((100vw - 40px) / 274px)) }` — fit-to-width scale, capped at 1.0.
- `.stage` — fixed 274 × 561 pt, scaled with `transform: scale(var(--s))`.
- `.viewport` — `calc(274px * var(--s))` × `calc(561px * var(--s))`, clips the scaled stage.
- `.page` — single centred column.
- Sheet styles: `.numpad-*`, `.alpha-*`, `.export-*`; shared `.numpad-backdrop`. The PROG 03 picker uses `.r3-picker` / `.entry-select` (a native `<select>` over the R3 register), not a sheet.

### 2. HTML (in `<body>`)

- `.panel-wrap` → `.panel` → `.viewport` → `.stage` → `.board`.
- `.board` stacks `.display` (bezel + screen) and `.keypad` (grid + four corner screws) with a 25 pt gap.
- Two decorative top screws sit in `.panel-wrap`, level with the top of the display.
- `.caption` sits below the stage.
- Hidden `#load-input` (`type="file"`) backs the LOAD action.
- Sheets: `#numpad-sheet` (PROG), `#alpha-sheet` (R1 A / R1 B), `#export-sheet` (OFFL Save / Cancel), plus two modals created on demand — constant/trail (PROG 02) and new file (PROG 04, also chained from PROG 01 when no file is resident). The PROG 03 picker is `#r3-picker` — a native `<select>` overlaid on the R3 register row — not a sheet.

### 3. JavaScript (in `<script>`)

One IIFE. Section order in the file:

1. 7-segment polygon data (`DIGITS`, `GLYPHS`) and SVG helpers (`makeSVG`, `renderTwoDigits`).
2. Gesture blocking, `PRESS_MS`, sound pools (`VOICES = 3`).
3. DOM refs and display state.
4. Renderers (`renderR1`, `renderR2`, `renderRx`, `clearAllFields`) and COMP REDY sync.
5. Resident-file lifecycle (`loadEntry`, `openExportSheet`, `performExport`, `unloadFile`, `saveCurrent`). OFFL's export chain: `showSaveFilePicker` (Chromium, Finder dialog) → `navigator.share({ files })` (iOS **Save to Files**) → anchor download fallback.
6. Message box (`messageState`, `updateMessage`) — see `.subagents/messages.md`.
7. Sheets and pickers: R3 entry picker (native `<select>`), numpad, R1 alpha, constant/trail, new file.
8. Clipboard helpers (`copyText`, `fallbackCopy`).
9. `handleKey` and keypad event wiring.

## Layout Map

```
┌──────────────────────────────────────────────┐
│ ┌───────────────────────┐                    │
│ │ ┌─────────────┐       │  ┌─────────────┐   │
│ │ │ COMP        │       │  │    PROG     │   │
│ │ │ REDY        │       │  │     88      │   │
│ │ └─────────────┘       │  └─────────────┘   │
│ │                       │                    │
│ │                       │                    │
│ │  ┌─────────────┐      │  ┌─────────────┐   │
│ │  │    VERB     │      │  │    NOUN     │   │
│ │  │     88      │      │  │     88      │   │
│ │  └─────────────┘      │  └─────────────┘   │
│ │                       │                    │
│ │ ───────────────────────────────────────────│
│ │ R1   + 88888                               │
│ │                                            │
│ │ ───────────────────────────────────────────│
│ │ R2   + 88888                               │
│ │                                            │
│ │ ───────────────────────────────────────────│
│ │ R3   + 88888                               │
│ │                                            │
│ └────────────────────────────────────────────┘
```

R3 doubles as the PROG 03 entry picker: `#r3-picker` absolutely overlays the R3 register cells (`.reg` is its positioning context) with a native `<select id="entry-select">`, hidden until PROG 03 reveals it.

## State and Data Model

The full contract, including every PROG/Save/Load flow, is in `.subagents/workflows.md`. Summary:

| State | Meaning |
|---|---|
| `displayState` | `{ prog, verb, noun, r1, r2, r3 }`; single source of truth for rendered values |
| `loadedLog` | Resident document `{ log, constant, trail }`, or `null` when no file is resident |
| `currentEntry` | Entry selected via PROG 03, or `null`; cleared after SAVE |
| `dirty` | Resident document differs from what was last saved |
| `constant` / `trail` | 12-character constant and 2-character trail stored on the document |
| `fileName` | Name used when OFFL exports the resident document |
| `messageState` | Drives the `.caption` message box — 9 status/error states, some timed. See `.subagents/messages.md` |

Resident state is in memory only; nothing is persisted between reloads, and SAVE never downloads.

## Component Geometry

All values derive from `ContentView.swift / DisplayPanelView / KeyView / SevenSegmentView`.

### Stage and Panel

| Element | Value |
|---|---|
| Stage | 274 × 561 pt |
| Panel padding | 32 pt top, 40 pt sides, 26 pt bottom (all × `--s`) |
| Board gap | 25 pt between display and keypad |
| Top panel screws | 22 pt, top `21 × --s`, left/right `40 × --s` |

### Display (`.display`, 201 × 341 pt)

| Element | Size | Position | Style |
|---|---|---|---|
| Bezel | 201 × 341 | inset 0 | fill `#b8b7bb`, 4 pt inner stroke `#29292e`, radius 9 |
| Screen | 189 × 329 | left 6, top 6 | gradient `#555342 → #77765a`, 4 pt inner stroke `#262622`, radius 5 |
| COMP REDY | 70 × 70 | left 7, top 7 | radius 4; 10.08 px / 12 px bold |
| Dot 1–3 | 6 pt ⌀ | left 91; top 12 / 76 / 139 | fill `#e7e5dc`, 40 % centre `#fffdf5` |
| PROG block | 59.5 wide | left 118, top 6 | pill + 2 digit cells |
| VERB block | 59.5 wide | left 14, top 86 | pill + 2 digit cells |
| NOUN block | 59.5 wide | left 108, top 86 | pill + 2 digit cells |
| Separator bars | 173 × 1.4 | left 8; top 148 / 204 / 260 | fill `#35d39a`, radius 0.7 |
| Register rows | — | left 8; top 153 / 208 / 263 | label + sign cell + 5 cells |

The R3 row's `.reg` is `position: relative`; `#r3-picker` fills it (`inset: 0`) with the native entry-select control (mint outline `rgba(53, 211, 154, 0.45)`, chevron drawn as an inline SVG data-URI in `#35d39a`).

### Cells

- Digit cell: 26 × 30 pt.
- Register cell: 26 × 30 pt; sign cell 12.1 × 30 pt; gap 1.5 pt; register padding 3 pt vertical.
- Register label: 8 pt wide, 7 px bold, colour `#9fb3a4`.
- Label pill: 59.5 × 12 pt, radius 1.5 pt, 7.5 px bold.
- Seven-segment glyphs are SVG `<polygon>` elements with exact normalized vertices from `SevenSegmentView`. Never replace them with a font.

### Keypad

| Element | Value |
|---|---|
| Grid | 3 columns × 2 rows, 68 pt cells |
| Column gap | 9 pt |
| Row gap | 7 pt |
| Padding (for screws) | 26 pt all sides |
| Key height | 68 pt; radius `width × 0.075` |
| Screws | 22 pt, four keypad corners |

Keypad screw rotations: TL −28°, TR +74°, BL +35°, BR −12°.

### Keys

| Label | Class | `data-key` | `data-sound` | Width |
|---|---|---|---|---|
| VERB | `.verb` | `VERB` | `5` | 68 px |
| CLR | `.clr` | `CLR` | `5` | 65 px |
| LOAD / OFFL | `.entr` | `ENTR` | `7` | 64 px |
| NOUN | `.noun` | `NOUN` | `5` | 68 px |
| PROG | `.stby` | `PROG` | `5` | 65 px |
| SAVE | `.rset` | `SAVE` | `5` | 64 px |

Press animation: `transform: scale(0.93)`, `transition: 70ms ease-out`, release after `PRESS_MS = 90`.

## Color Tokens

| Token | Value | Used for |
|---|---|---|
| Mint (`DSKY.Display`) | `#35d39a` | Segment fill, separator bars, pill background, COMP REDY ready |
| Pill text | `#101b16` | Label pill foreground, COMP REDY text when ready |
| Bezel fill | `#b8b7bb` | Display bezel background |
| Bezel stroke | `#29292e` | 4 pt inner border |
| Screen top | `#555342` | Screen gradient start |
| Screen bottom | `#77765a` | Screen gradient end |
| Screen stroke | `#262622` | 4 pt inner border |
| Register label | `#9fb3a4` | R1/R2/R3 labels, COMP REDY text when off |
| COMP REDY entry | `#ff9f0a` | Unchanged entry loaded |
| COMP REDY dirty | `#e02b2b` | Unsaved entry or settings |

## Sound System

Three WAV clips exist in `sd/`. They are played through `<Audio>` elements (not Web Audio) so `file://` works.

| Clip | Used by |
|---|---|
| `sd/4.wav` | Nothing — retained asset, not referenced by any `data-sound` |
| `sd/5.wav` | VERB, CLR, NOUN, PROG, SAVE |
| `sd/7.wav` | LOAD / OFFL |

Each clip has a round-robin pool of 3 `Audio` instances (`VOICES = 3`), so rapid presses overlap instead of restarting.

## Supported Environments

- Any modern browser: Safari, Chrome, Firefox (desktop or mobile).
- Works from `file://`; no server required.
- Fit-to-width scaling caps at 1.0× on wide screens.
- No dependencies, no bundler, no npm, no build step.

## Documentation Map

| Document | Owns |
|---|---|
| `AGENTS.md` | Launch-critical constraints, verification, quick reference |
| `.subagents/overview.md` | Architecture, file structure, geometry, colour and sound tokens |
| `.subagents/workflows.md` | Runtime state, key behaviour, sheets, JSON contract |
| `.subagents/messages.md` | Message box states, triggers, timing, integration points |
| `.subagents/CHANGELOG.md` | Removed behaviour and historical changes |
| `.subagents/documentation.md` | Documentation rules and enforcement |
