# Pixel Painter

A minimal, zero-dependency pixel art editor that runs entirely in the browser.

![HTML5](https://img.shields.io/badge/HTML5-Canvas-orange)
![No Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)

## Features

- **Paint & Erase** — Left-click to paint, right-click to erase
- **16-Color Palette** — Classic PICO-8 color palette
- **Infinite Canvas** — Pan with middle-click drag or scroll wheel
- **Auto-Save** — Work is saved to localStorage automatically
- **Smooth Strokes** — Uses coalesced pointer events for fluid drawing
- **Single File** — No build step, no dependencies, just open and draw

## Getting Started

Open `index.html` in any modern browser. That's it.

```bash
# Or serve it locally
npx serve .
```

## Controls

| Action     | Input                               |
| ---------- | ----------------------------------- |
| Paint      | Left-click / drag                   |
| Erase      | Right-click / drag                  |
| Pan        | Middle-click drag or scroll wheel   |
| Pick color | Click the palette bar at the bottom |

## How It Works

The entire app is a single HTML file (~190 lines) using the Canvas API:

- **Grid** — Cells are 24x24 screen pixels on an infinite coordinate plane
- **Storage** — Cell data is stored as a `Map` of `"x,y"` keys to color indices, serialized to `localStorage` as JSON
- **Rendering** — A `requestAnimationFrame` loop redraws only when state changes (dirty flag). Filled cells are batched by color to minimize draw calls
- **Palette** — The selection indicator automatically picks a black or white outline based on the selected color's luminance for contrast

## Palette

The 16 colors come from the [PICO-8](https://www.lexaloffle.com/pico-8.php) palette:

|           |           |           |           |           |           |           |           |           |           |           |           |           |           |           |           |
| --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- | --------- |
| `#000000` | `#1D2B53` | `#7E2553` | `#008751` | `#AB5236` | `#5F574F` | `#C2C3C7` | `#FFF1E8` | `#FF004D` | `#FFA300` | `#FFEC27` | `#00E436` | `#29ADFF` | `#83769C` | `#FF77A8` | `#FFCCAA` |

## Browser Support

Works in all modern browsers that support:

- Canvas 2D
- Pointer Events
- `getCoalescedEvents()` (gracefully degrades without it)
- localStorage

## License

MIT
