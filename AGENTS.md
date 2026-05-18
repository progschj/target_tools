# agents.md

## Project Overview
Zero-dependency single-page app for ISSF 25m Precision / 50m Pistol target practice. Renders an official-spec target with click/tap shot marking and a "Punch In" zoom that frames the 7-ring to the smaller viewport dimension. Offline-capable, mobile-optimized, pixel-perfect coordinate mapping at any zoom level.

## Tech Stack
| Category | Technology |
|----------|------------|
| Language | Vanilla HTML5, CSS3, JavaScript (ES6+) |
| Rendering | SVG (inline, viewBox="0 0 550 550") |
| Zoom | CSS transform: scale() + transform-origin: center |
| Coordinates | SVGPoint.matrixTransform(svg.getScreenCTM().inverse()) |
| Dependencies | None |
| Format | Single index.html |

## Core Features
- Full-screen responsive target, 1:1 aspect ratio
- Click/tap to place red cross markers at exact shot locations
- Double-tap or button toggle for "Punch In" zoom (7-ring fills smaller axis)
- Shot counter and clear/reset controls
- Touch-optimized (touch-action: none, double-tap debounce)
- Pixel-perfect coordinates across all zoom states and device pixel ratios
- Offline, zero build step

## Architecture
```
issf-25m-target/
└── index.html          # Single self-contained file (HTML + CSS + JS + SVG)
```

```
<body>
  <div class="app-wrapper">
    <div class="ui-overlay">          # Non-interactive UI layer
      <div class="status">...</div>
      <div class="hint">...</div>
      <div class="controls">...</div>
    </div>
    <svg id="target">...</svg>        # Target + shot markers
  </div>
</body>
```

## Key Implementation Details

### Coordinate Mapping (Critical)
Never use getBoundingClientRect() + manual scaling. It breaks with CSS transforms.

```js
function getSVGPoint(event) {
  const point = svg.createSVGPoint();
  point.x = event.touches ? event.touches[0].clientX : event.clientX;
  point.y = event.touches ? event.touches[0].clientY : event.clientY;
  return point.matrixTransform(svg.getScreenCTM().inverse());
}
```
This handles viewBox scaling, CSS transforms, and device pixel ratio automatically.

### Zoom Logic
- Scale: `Math.min(window.innerWidth, window.innerHeight) / 200` (7-ring diameter = 200mm)
- `transform-origin: center center` + `transform-box: fill-box` for consistent centering
- Transition: `transform 0.35s cubic-bezier(0.25, 0.1, 0.25, 1)`

### Touch Handling
```js
svg.addEventListener('touchstart', (e) => {
  const now = Date.now();
  if (now - lastTap < 300) { e.preventDefault(); toggleZoom(); return; }
  lastTap = now;
  e.preventDefault();
  addCross(...getSVGPoint(e));
}, { passive: false });
```
300ms debounce for single-tap (mark) vs double-tap (zoom). `passive: false` + `touch-action: none` prevents gesture conflicts.

### SVG Layering
- Draw order: Background -> Black inner zone -> Rings -> Markers
- Markers: `pointer-events: none`, red cross 1.5px stroke, 12px diameter

## Guidelines
| Rule | Reason |
|------|--------|
| Maintain single-file structure | Portability & offline use |
| Never replace getScreenCTM().inverse() with getBoundingClientRect() | Manual scaling breaks under CSS transforms & DPR |
| Preserve transform-origin: center + transform-box: fill-box | Prevents zoom offset on iOS/Android |
| Keep touch-action: none + passive: false | Blocks native pinch/scroll conflicts |
| Use CSS variables for colors/sizes if extending | Maintainability |
| Test on iOS Safari & Android Chrome | Gesture APIs vary |
| No external dependencies | Zero-dependency requirement |

## Known Limitations
- No persistent state (refresh clears shots)
- DOM markers accumulate; clearBtn removes them but doesn't enforce limits
- Double-tap may need two quick taps on some Android devices
- No auto-scoring, shot grouping, or MOA calculation
- Uses max-width/max-height instead of object-fit: contain (SVG transform conflicts)

## Future Enhancements
1. Scoring engine: map shot coordinates to 1-10 ring values by distance from center
2. Export: PNG/PDF with shot markers
3. Persistence: localStorage shot history with timestamps
4. Accessibility: ARIA labels, keyboard navigation, high-contrast mode
5. Multi-stage: toggle between precision/rapid-fire layouts
6. Analytics: shot clustering, dispersion ellipses, trend graphs

## ISSF Compliance
- Dimensions per ISSF Rule Book 2022-2028, Section 7.3.1.2
- 10-ring: 50mm | Inner ten: 25mm | 7-ring: 200mm diameter
- Black inner zone (r <= 100mm), white rings 7-10 + inner ten
- Numbers 7-9: white | Numbers 1-6: black | 10-zone: unnumbered
- Alignment markers: red crosses at 25mm from card edges
- Ring thickness: 0.3px stroke

## Usage
- AI Agents: Reference before generating, modifying, or debugging. Follow guidelines strictly.
- Developers: Use as maintenance checklist and extension roadmap.
- Reviewers: Validate against ISSF compliance and coordinate mapping rules.

Tip: When adding features, verify coordinates with `console.log(getSVGPoint(e))` at both zoom levels.
