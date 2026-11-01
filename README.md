# DSKY HTML App for iPhone 13

A responsive HTML/CSS/JavaScript implementation of the Apollo Guidance Computer DSKY interface optimized for iPhone 13 dimensions.

## Features

- **iPhone 13 Optimized**: Designed for 390x844 points (1170x2532 pixels @3x)
- **Responsive Design**: Works on all iPhone sizes with proper scaling
- **DSKY Authentic Styling**: 
  - Olive green display background
  - Dark gradient button styling
  - Authentic DSKY color palette
  - Seven-segment display style typography
- **Interactive Elements**:
  - 6-button layout (VERB, NOUN, ENTER, CLR, PRO, RSET)
  - Visual feedback on button press
  - Audio feedback (Web Audio API)
  - Blinking cursor for entry mode
- **State Management**: 
  - PROG/VERB/NOUN displays
  - R1/R2/R3 registers
  - Entry buffer system
  - Mode management (verb/noun entry)

## File Structure

```
html/
├── index.html          # Main HTML app
├── iosDSKY.jpg         # Reference layout image
└── README.md           # This documentation
```

## Layout Details

### Display Area
- **PROG**: Program number display (shows "88" on startup)
- **VERB**: Verb entry display (2 digits, blinks when awaiting input)
- **NOUN**: Noun entry display (2 digits, blinks when awaiting input)
- **R1**: Register 1 (5 digits, shows distance in KM)
- **R2**: Register 2 (5 digits, shows distance in M)
- **R3**: Register 3 (5 digits, shows distance in KM)

### Button Grid (3×2)
1. **VERB** - Select verb entry mode
2. **NOUN** - Select noun entry mode  
3. **ENTER** - Execute/confirm entry (green)
4. **CLR** - Clear current entry
5. **PRO** - Proceed/continue (placeholder)
6. **RSET** - Reset all entries (red)

## Implementation Notes

### Design Decisions
1. **Simplified Layout**: Based on reference image `iosDSKY.jpg`, showing display + 6 buttons
2. **Color Scheme**: Matches DSKY hardware palette from SVG reference
3. **Typography**: Monospace for displays, sans-serif for labels
4. **Interactions**: 
   - Visual button press feedback
   - Audio click sound
   - Blinking entry indicators
5. **Responsive**: Uses CSS Grid and Flexbox for layout

### Technical Details
- **Viewport**: `width=device-width, initial-scale=1.0` for proper mobile scaling
- **CSS**: Custom properties, gradients, shadows for depth
- **JavaScript**: ES6 class for state management
- **Audio**: Web Audio API for button feedback
- **Touch Events**: Optimized for mobile interaction

## Usage

1. Open `index.html` in any modern browser
2. On iPhone 13: Open directly in Safari
3. **Testing**: 
   - Tap buttons to interact
   - VERB/NOUN displays blink when awaiting input
   - ENTER confirms entries
   - CLR clears current entry
   - RSET resets all

## Development Notes

### Ready for Command Implementation
The app structure is complete with:
- Visual layout matching DSKY aesthetic
- State management system
- Event handling for all buttons
- Placeholder logic for future command implementation

### To Extend:
1. Add number pad (0-9, +, -) if needed
2. Implement specific verb/noun logic
3. Add data persistence
4. Connect to backend APIs
5. Add advanced display modes

## References
- Original DSKY SVG: `Documents/img/DSKY.svg`
- Reference layout: `html/iosDSKY.jpg`
- AGENTS.md: Project constraints and design rules

## Compatibility
- iPhone 13/13 Pro/13 Pro Max/13 Mini
- All modern iOS browsers
- Desktop browsers (responsive scaling)
- iPad (scaled appropriately)