# Improvement Plan

## 1. Code Organization

### Extract Configuration
Move magic numbers to a config object at the top of the script:
```javascript
const CONFIG = {
    TOLERANCE_CM: 15,
    VERTICAL_MARGIN_CM: 15,
    MIN_BASE_DIFFERENCE: 5,
    MAX_SPIRAL_ITERATIONS: 100,
    CANVAS_WIDTH: 800,
    CANVAS_HEIGHT: 600,
    SCALE_PADDING: 1.3  // currently hardcoded in line 252
};
```

### Modularize Functions
Split into logical modules if the file grows:
- `geometry.js` - syncGeometry, isPointInPoly, distToSegment, checkRectFit
- `renderer.js` - draw function, canvas operations
- `ui.js` - DOM element management, event listeners

## 2. Performance Optimizations

### Debounce Input Events
Currently every slider movement triggers full redraw. Add debouncing:
```javascript
function debounce(fn, delay = 16) {
    let timeout;
    return (...args) => {
        clearTimeout(timeout);
        timeout = setTimeout(() => fn(...args), delay);
    };
}
```

### Cache Polygon Calculations
The trapezoid polygon is recalculated on every draw. Cache it when geometry changes and only recalculate rectangles when detail dimensions change.

### Early Exit in Packing Loop
Add bounds check before calling `checkRectFit` - if rectangle X position is clearly outside trapezoid bounds, skip the expensive collision check.

## 3. User Experience

### Add Numeric Input Fields
Allow direct number entry alongside sliders for precise values:
```html
<input type="number" id="inputANum" min="200" max="1200" value="500">
```

### Add Undo/Redo
Store state history to allow reverting changes.

### Export Functionality
- Export layout as PNG/SVG
- Export measurements as JSON/CSV for CNC machines

### Keyboard Shortcuts
- Arrow keys to fine-tune selected parameter
- Number keys to quickly select presets

### Presets
Add common trapezoid configurations as quick-select buttons.

## 4. Algorithm Improvements

### Alternative Packing Strategies
Current greedy spiral may not be optimal. Consider:
- Bottom-left fill algorithm
- Guillotine cutting approach
- Allow rotation (90°) of rectangles for better fit

### Gap Analysis
Show wasted space visually and suggest better detail dimensions.

### Multi-Detail Support
Allow packing multiple different rectangle sizes in one trapezoid.

## 5. Code Quality

### Add Input Validation
```javascript
function validateInput(value, min, max, defaultVal) {
    const num = parseInt(value);
    if (isNaN(num)) return defaultVal;
    return Math.max(min, Math.min(max, num));
}
```

### Error Handling
Wrap canvas operations in try-catch, show user-friendly error messages.

### Use Constants for Colors
```javascript
const COLORS = {
    TRAPEZOID_FILL: '#f8fafc',
    TRAPEZOID_STROKE: '#2563eb',
    RECT_FULL: { fill: 'rgba(16, 185, 129, 0.4)', stroke: '#10b981' },
    RECT_TOLERANCE: { fill: 'rgba(245, 158, 11, 0.4)', stroke: '#f59e0b' }
};
```

## 6. Accessibility

### Add ARIA Labels
```html
<input type="range" id="inputA" aria-label="Bottom base length in centimeters">
```

### Keyboard Navigation
Ensure all controls are keyboard accessible with visible focus states.

### High Contrast Mode
Add toggle for users with visual impairments.

## 7. Internationalization

### Extract Strings
Move all Bulgarian text to a translations object for easy localization:
```javascript
const i18n = {
    bg: {
        title: 'Разкрояване с допустимо излизане',
        bottomBase: 'Основа a (Долна)',
        // ...
    },
    en: {
        title: 'Cutting with Allowable Overhang',
        bottomBase: 'Base a (Bottom)',
        // ...
    }
};
```

## 8. Testing

### Add Unit Tests
Test critical functions:
- `isPointInPoly` - various polygon shapes and edge cases
- `checkRectFit` - boundary conditions, tolerance edge cases
- `distToSegment` - perpendicular and endpoint cases
- `syncGeometry` - constraint enforcement

### Visual Regression Tests
Capture canvas snapshots for known inputs to detect rendering changes.

## Priority Order

| Priority | Task | Effort | Impact |
|----------|------|--------|--------|
| 1 | Extract configuration | Low | Medium |
| 2 | Debounce inputs | Low | Medium |
| 3 | Add numeric input fields | Low | High |
| 4 | Export PNG/SVG | Medium | High |
| 5 | Add unit tests | Medium | High |
| 6 | Rectangle rotation option | Medium | High |
| 7 | Internationalization | Medium | Medium |
| 8 | Multi-detail support | High | High |