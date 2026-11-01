# overview.md — DSKY Project Architecture

## Project Orientation

DSKY is a standalone HTML/CSS/JavaScript analysis artifact — not part of any Xcode project or iOS build. It reproduces the Apollo Guidance Computer DSKY interface in a browser, for visual inspection and study. All geometry, proportions, colors, and segment shapes are derived from a Swift/SwiftUI source (`ContentView.swift`). There is no backend, no bundler, and no build step.

Open `7.html` directly in any modern browser (`file://` works). Audio plays after the first user gesture.

## File Structure

```
DSKY/
├── 7.html                  ← Current artifact: display panel + 3×2 keypad + analysis sidebar
├── 6.html                  ← Predecessor: 3×2 keypad + screws only (no display panel)
├── entr.html               ← Single-key study: ENTR key in isolation
├── sd/
│   ├── 4.wav               ← Click sound for left column (VERB, NOUN)
│   ├── 5.wav               ← Click sound for middle column (CLR, STBY)
│   └── 7.wav               ← Click sound for right column (ENTR, RSET)
├── img/
│   └── screw.png           ← Screw image (22 pt, used at 4 corners of keypad)
├── mockups/
│   └── iosDSKY.jpg         ← Reference layout image. Never delete.
├── screenshots/            ← Reference screenshots. Never delete.
├── .subagents/
│   ├── documentation.md    ← Documentation organization and enforcement rules
│   └── overview.md         ← This file
└── AGENTS.md               ← Launch-critical rules and quick reference
```

## Architecture

`7.html` is a single self-contained file. It has three logical sections:

### 1. CSS (in `<style>`)

All layout and visual styles are inline. Key rules:

- `:root { --s: 1.2 }` — uniform stage scale. Changing `--s` scales the entire stage.
- `.page` — two-column grid (panel + analysis). Collapses to single column below 1000 px.
- `.stage` — fixed 274 × 561 pt, scaled via `transform: scale(var(--s))`.
- `.viewport` — computed wrapper `calc(274px * var(--s))` × `calc(561px * var(--s))` that provides scroll bounds.

### 2. HTML (in `<body>`)

Two top-level sections inside `.page`:

**`<main class="panel">`** — the DSKY hardware replica:
- `.display` → `.bezel` + `.screen`: display panel with COMP ACTY, dots, PROG/VERB/NOUN two-digit groups, separator bars, R1/R2/R3 register rows.
- `.keypad` → `.grid`: 3×2 button grid + 4 corner screw images.

**`<aside class="analysis">`** — interactive controls:
- Scale range input (0.5×–2.4×, default 1.2×).
- "Hold pressed" toggle — freezes all keys in pressed state.
- "Sound on/off" toggle.
- Per-key activation count display (6 buttons × count).
- Spec readout table (mirrors key dimensions from the Swift source).

### 3. JavaScript (in `<script>`)

Single IIFE. Key constants and state:

| Identifier | Value / purpose |
|---|---|
| `PRESS_MS` | `90` — activation flash duration (ms); from `KeyView asyncAfter(.now() + 0.09)` |
| `VOICES` | `3` — audio voices per clip for rapid-overlap playback |
| `sounds` | Map of `{4, 5, 7}` → `{voices: Audio[], next: int}` round-robin pools |
| `soundOn` | Boolean, toggled by sound button |
| `counts` | Object keyed by `data-key`, per-button activation counter |

Event flow per button press:
1. `pointerdown` → `press(k)` (adds `.is-pressed`) + `playFor(k)` (fires sound immediately).
2. `pointerup` → `activate(k)` (increments count, calls `press(k)`, then `setTimeout(release, 90)`).
3. `pointercancel` / `pointerleave` (no buttons held) → `release(k)`.
4. Keyboard `click` with `event.detail === 0` → `activate(k)` + `playFor(k)`.

Hold toggle: sets `aria-pressed="true"` on `#holdBtn`; `release()` checks this and is suppressed when active.

## Component Geometry

All values are from `ContentView.swift / DisplayPanelView / KeyView / SevenSegmentView`.

### Display Panel (`.display`, 201 × 341 pt)

| Element | Size | Position | Style |
|---|---|---|---|
| Bezel | 201 × 341 | inset: 0 | fill `#b8b7bb`, 4 pt inner stroke `#29292e`, radius 9 |
| Screen | 189 × 329 | left 6, top 6 | gradient `#555342 → #77765a`, 4 pt stroke `#262622`, radius 5 |
| COMP ACTY label | 52 × 42 | left 8, top 8 | 10.08 px bold, color `#2a3a2e` (unlit) |
| Dot 1 | 6 pt ⌀ | left 91, top 12 | fill `#e7e5dc`, 40 % centre `#fffdf5` |
| Dot 2 | 6 pt ⌀ | left 91, top 76 | same |
| Dot 3 | 6 pt ⌀ | left 91, top 139 | same |
| PROG two-digit | 59.5 × — | left 118, top 6 | mint pill + 2 × digit cells |
| VERB two-digit | 59.5 × — | left 14, top 86 | mint pill + 2 × digit cells |
| NOUN two-digit | 59.5 × — | left 108, top 86 | mint pill + 2 × digit cells |
| Separator bar 1 | 173 × 1.4 | left 8, top 148 | fill `#35d39a`, radius 0.7 |
| Separator bar 2 | 173 × 1.4 | left 8, top 204 | same |
| Separator bar 3 | 173 × 1.4 | left 8, top 260 | same |
| R1 register row | — | left 8, top 153 | label + sign cell + 5 digit cells |
| R2 register row | — | left 8, top 208 | same |
| R3 register row | — | left 8, top 263 | same |

### Digit Cell (26 × 30 pt)

Seven-segment glyphs rendered as SVG `<polygon>` elements with exact normalized vertex coordinates from `SevenSegmentView`. Fill color `#35d39a`. Do not replace with a font or icon.

### Register Cell

- Standard digit cell: 22 × 25 pt.
- Sign cell: 12.1 × 25 pt.
- Contained in `.reg` (padding 3 px top/bottom, 4 px sides; gap 1.5 pt).

### Label Pill

- Size: 59.5 × 12 pt (including 1.5 px vertical padding).
- Background: `#35d39a`, text `#101b16`, 7.5 px bold, radius 1.5 pt.

### Keypad (`.keypad`)

| Element | Value |
|---|---|
| Grid | 3 columns × 2 rows, 68 pt cells |
| Column gap | 9 pt |
| Row gap | 7 pt |
| Padding (for screws) | 26 pt all sides |

### Keys

| Key | CSS class | Width (`--w`) | Color stops (rest) | Color stops (pressed) |
|---|---|---|---|---|
| VERB | `.verb` | 68 px | `#0a1e0a` → `rgba(10,30,10,.6)` | same @ 60% → same @ 36% |
| NOUN | `.noun` | 68 px | same as VERB | same |
| CLR | `.clr` | 65 px | `#101015` → `#030306` | same @ 60% |
| STBY | `.stby` | 65 px | same as CLR | same |
| ENTR | `.entr` | 64 px | `#1e0a0a` → `rgba(30,10,10,.6)` | same @ 60% → same @ 36% |
| RSET | `.rset` | 64 px | same as ENTR | same |

Press animation: `transform: scale(0.93)`, `transition: 70ms ease-out`.

### Screws

- `img/screw.png`, 22 pt diameter, border-radius 50 %, at four corners of `.keypad`.
- Each rotated independently: TL −28°, TR +74°, BL +35°, BR −12°.

## Color Tokens

| Token | Value | Used for |
|---|---|---|
| Mint (`DSKY.Display`) | `#35d39a` | Segment fill, separator bars, label pill background, scale slider accent |
| Pill text | `#101b16` | Label pill foreground |
| Bezel fill | `#b8b7bb` | Display bezel background |
| Bezel stroke | `#29292e` | 4 pt inner border on bezel |
| Screen top | `#555342` | Screen gradient start |
| Screen bottom | `#77765a` | Screen gradient end |
| Screen stroke | `#262622` | 4 pt inner border on screen |
| COMP ACTY unlit | `#2a3a2e` | COMP ACTY label (no backlight) |
| Register label | `#9fb3a4` | R1/R2/R3 text color |

## Sound System

Three WAV clips in `sd/`, one per key column. Played via `<Audio>` elements (not Web Audio API) to ensure `file://` compatibility.

| Clip | Column | Keys |
|---|---|---|
| `sd/4.wav` | Left | VERB, NOUN |
| `sd/5.wav` | Middle | CLR, STBY |
| `sd/7.wav` | Right | ENTR, RSET |

Each clip has a round-robin pool of 3 `Audio` instances (`VOICES = 3`), allowing rapid overlapping presses without restarting the clip.

## Supported Environments

- Any modern browser: Safari, Chrome, Firefox (desktop or mobile).
- Works from `file://` — no server needed.
- Responsive: single-column layout below 1000 px viewport width.
- No dependencies, no bundler, no npm, no build step.
