# AGENTS.md — DSKY Project Guide

## Start Here

DSKY is a standalone HTML/CSS/JavaScript analysis artifact reproducing the Apollo Guidance Computer DSKY interface. It is **not** an Xcode project. The current artifact (`7.html`) renders the full display panel plus a 3×2 function keypad, with an analysis sidebar for interactive inspection. All geometry and colors are derived from a Swift/SwiftUI source (`ContentView.swift / DisplayPanelView / KeyView / SevenSegmentView`).

## Working Rules

- Progress visibility: `Progress: analyzing codebase…`, `Progress: generating patch…`, `Progress: running verification…`
- Never silent >10s during internal work.
- Explain: non-engineer summary first, then user-visible consequences grouped by severity, then implementation details.
- **Vision gate (0):** attached image not visible to model → stop and notify. Never infer.

## Build and Verify

- Requirements: any modern browser (Safari, Chrome, Firefox). No build step.
- Entry point: open `7.html` directly in a browser (`file://` works; Audio will play after a user gesture).
- Assets required at runtime: `img/screw.png`, `sd/4.wav`, `sd/5.wav`, `sd/7.wav`.
- No automated tests. Verify visually: display panel, keypad, scale slider, hold toggle, sound toggle all respond correctly.
- File tree: `7.html` (current), `6.html` (keypad-only predecessor), `entr.html` (single-key study). See `.subagents/overview.md` for full structure.

## Critical Constraints

0. **Vision gate** — image not visible to model → stop, notify. Never infer.
1. All geometry (sizes, positions, radii) must match the Swift source values documented in `7.html`'s header comment and in `.subagents/overview.md`. Do not approximate.
2. Segment polygon vertices are exact normalized coordinates from `SevenSegmentView`. Never replace them with a font.
3. Mint accent color is `#35d39a` (DSKY.Display token). Do not use any other value.
4. Sound uses `<Audio>` elements, not Web Audio API, so `file://` loads work. Do not switch to `fetch()` or `AudioContext` for the click sounds.
5. Three voices per clip (`VOICES = 3`) allow rapid overlapping presses. Do not reduce to a single `<audio>` element.
6. Press animation: `scale(0.93)`, 70 ms `ease-out`, 90 ms flash (`PRESS_MS`). Preserve both durations.
7. Scale is a CSS custom property `--s` on `:root`; the `.stage` uses `transform: scale(var(--s))`. Do not restructure this.
8. `7.html` is self-contained (single file, no bundler). Keep it that way unless explicitly asked to split.
9. This is an analysis artifact — do not treat it as production code or add a backend, auth, or data persistence unless asked.
10. `mockups/iosDSKY.jpg` and `screenshots/` are reference images only. Never delete them.

## Runtime Rules (see `.subagents/overview.md`)

- Button `data-sound` values map to clip files: `"4"` → `sd/4.wav`, `"5"` → `sd/5.wav`, `"7"` → `sd/7.wav`.
- Sound column mapping: left (VERB/NOUN) → 4.wav; middle (CLR/STBY) → 5.wav; right (ENTR/RSET) → 7.wav.
- Pointer-down fires the sound; pointer-up fires the activation (count increment + press flash).
- Keyboard activation fires on `click` with `event.detail === 0`; pointer path does not re-sound.
- "Hold pressed" toggle freezes all keys in pressed state for visual inspection; release is suppressed until toggled off.

## Quick Reference

| Setting | Value |
|---|---|
| Current artifact | `7.html` |
| Stage size | 274 × 561 pt |
| Bezel | 201 × 341, radius 9 |
| Screen | 189 × 329, radius 5 |
| Digit cell | 26 × 30 pt |
| Register cell | 22 × 25 pt |
| Key grid | 3 × 2, 68 pt cells, 7 pt row / 9 pt col gap |
| Screws | 22 pt diameter, 4 corners |
| Mint token | `#35d39a` |
| Press scale | 0.93, 70 ms ease-out, 90 ms flash |
| Sound voices | 3 per clip (4.wav / 5.wav / 7.wav) |
| Default scale | 1.2× |

## Documentation Rules

- Every source change updates `AGENTS.md` + affected `.subagents/` docs in the same commit.
- **AGENTS.md ≤ 150 lines** (excl. reference table). If exceeded, move detail to `.subagents/` and replace with pointer.
- Docs describe verified present implementation only. Deleted behavior → `.subagents/CHANGELOG.md`.
- New config keys → `.subagents/config.md`; public APIs → `.subagents/api.md`; historical changes → `.subagents/CHANGELOG.md`.

## Detailed Reference

| File | Use for |
|---|---|
| `.subagents/overview.md` | Architecture, file structure, component geometry, color tokens |
| `.subagents/documentation.md` | Documentation organization and enforcement rules |
| `.subagents/CHANGELOG.md` | Historical changes only (create when first needed) |
| `.subagents/config.md` | Configuration registry (create when first needed) |
