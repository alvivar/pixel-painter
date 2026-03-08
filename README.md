# Pixel Painter

A minimal, zero-dependency pixel art editor that runs entirely in the browser.

![HTML5](https://img.shields.io/badge/HTML5-Canvas-orange)
![No Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)

## Features

- **Paint & Erase** — Left-click to paint, right-click to erase
- **16-Color Palette** — Classic PICO-8 color palette
- **Eyedropper** — Alt+click to pick a color from the canvas
- **Infinite Canvas** — Pan with middle-click drag
- **Zoom** — Scroll wheel to zoom toward cursor (0.25×–8×), pinch-to-zoom on touch
- **Undo / Redo** — Full stroke-level undo (Ctrl+Z) and redo (Ctrl+Shift+Z)
- **Export PNG** — Export the viewport (Ctrl+E) or tight-cropped artwork (Ctrl+Alt+E)
- **Touch Support** — Pinch to zoom/pan; painting is suppressed during two-finger gestures
- **Auto-Save** — Work is saved to localStorage automatically (debounced)
- **Resume** — Re-opens centered on the last painted pixel
- **Smooth Strokes** — Uses coalesced pointer events for fluid drawing
- **Help Overlay** — Press `?` to see all keyboard shortcuts
- **Single File** — No build step, no dependencies, just open and draw

## Getting Started

Open `index.html` in any modern browser. That's it.

```bash
# Or serve it locally
npx serve .
```

## Controls

| Action          | Input                              |
| --------------- | ---------------------------------- |
| Paint           | Left-click / drag                  |
| Erase           | Right-click / drag                 |
| Eyedropper      | Alt + click                        |
| Pick color      | Click the palette bar              |
| Zoom            | Scroll wheel (zooms toward cursor) |
| Pan             | Middle-click drag                  |
| Pinch           | Two-finger pinch to zoom + drag    |
| Undo            | Ctrl+Z                             |
| Redo            | Ctrl+Shift+Z                       |
| Export viewport | Ctrl+E                             |
| Export artwork  | Ctrl+Alt+E                         |
| Help            | ?                                  |

## How It Works

The entire app is a single HTML file (~580 lines) using the Canvas API:

- **Grid** — Base cell size is 24px, scaled by a zoom factor (0.25×–8×). Grid lines auto-hide when cells are smaller than 4px to reduce visual noise
- **Storage** — Cells are stored in a `Map<string, {x, y, ci}>` with pre-parsed coordinates so the draw loop never parses strings. Saves are debounced (300ms) and serialized as streaming JSON — no intermediate array, no `JSON.stringify`
- **Rendering** — A `requestAnimationFrame` loop redraws only when state changes (dirty flag). Filled cells are batched by color to minimize `fillStyle` changes. Only on-screen cells are drawn
- **Undo/Redo** — Each paint/erase stroke is captured as a `Map<key, { before, after }>` delta. Undo replays `before` states, redo replays `after` states. The redo stack is cleared on new strokes
- **Export** — Viewport export captures the on-screen canvas minus the palette bar. Artwork export computes a tight bounding box and renders 1:1 pixel art to a temporary canvas
- **Palette** — Rendered to an offscreen canvas and blitted via `drawImage`; redrawn only on color change or resize. The selection indicator automatically picks a black or white outline based on luminance
- **Help Overlay** — Canvas-rendered shortcut panel driven by a declarative `HELP_BINDINGS` array. Uses PICO-8 palette colors for styling; dismisses on click or Escape
- **Touch** — Pinch-to-zoom and two-finger pan are handled via touch events. An `activeTouches` counter suppresses painting during multi-finger gestures

## Palette

The 16 colors come from the [PICO-8](https://www.lexaloffle.com/pico-8.php) palette:

|                                                                                      | Hex       | Name        |                                                                                      | Hex       | Name     |
| :----------------------------------------------------------------------------------: | --------- | ----------- | :----------------------------------------------------------------------------------: | --------- | -------- |
| ![](https://img.shields.io/badge/-%20%20-000000?style=flat-square&labelColor=000000) | `#000000` | Black       | ![](https://img.shields.io/badge/-%20%20-FF004D?style=flat-square&labelColor=FF004D) | `#FF004D` | Red      |
| ![](https://img.shields.io/badge/-%20%20-1D2B53?style=flat-square&labelColor=1D2B53) | `#1D2B53` | Dark Blue   | ![](https://img.shields.io/badge/-%20%20-FFA300?style=flat-square&labelColor=FFA300) | `#FFA300` | Orange   |
| ![](https://img.shields.io/badge/-%20%20-7E2553?style=flat-square&labelColor=7E2553) | `#7E2553` | Dark Purple | ![](https://img.shields.io/badge/-%20%20-FFEC27?style=flat-square&labelColor=FFEC27) | `#FFEC27` | Yellow   |
| ![](https://img.shields.io/badge/-%20%20-008751?style=flat-square&labelColor=008751) | `#008751` | Dark Green  | ![](https://img.shields.io/badge/-%20%20-00E436?style=flat-square&labelColor=00E436) | `#00E436` | Green    |
| ![](https://img.shields.io/badge/-%20%20-AB5236?style=flat-square&labelColor=AB5236) | `#AB5236` | Brown       | ![](https://img.shields.io/badge/-%20%20-29ADFF?style=flat-square&labelColor=29ADFF) | `#29ADFF` | Blue     |
| ![](https://img.shields.io/badge/-%20%20-5F574F?style=flat-square&labelColor=5F574F) | `#5F574F` | Dark Grey   | ![](https://img.shields.io/badge/-%20%20-83769C?style=flat-square&labelColor=83769C) | `#83769C` | Lavender |
| ![](https://img.shields.io/badge/-%20%20-C2C3C7?style=flat-square&labelColor=C2C3C7) | `#C2C3C7` | Light Grey  | ![](https://img.shields.io/badge/-%20%20-FF77A8?style=flat-square&labelColor=FF77A8) | `#FF77A8` | Pink     |
| ![](https://img.shields.io/badge/-%20%20-FFF1E8?style=flat-square&labelColor=FFF1E8) | `#FFF1E8` | White       | ![](https://img.shields.io/badge/-%20%20-FFCCAA?style=flat-square&labelColor=FFCCAA) | `#FFCCAA` | Peach    |

## Browser Support

Works in all modern browsers that support:

- Canvas 2D
- Pointer Events
- Touch Events (for pinch-to-zoom)
- `getCoalescedEvents()` (gracefully degrades without it)
- localStorage

## License

MIT
