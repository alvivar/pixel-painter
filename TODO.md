# TODO

## Features

- [x] Zoom support (scroll-zoom, pinch-to-zoom)
  - Add `scale`, `MIN_SCALE` (0.25), `MAX_SCALE` (8) state variables
  - Replace all `PX` usage with `PX * scale` in `draw()` and `cellAt()`
  - Wheel handler: zoom toward cursor (`pan = cursor - (cursor - pan) * (newScale / oldScale)`)
  - Pinch-to-zoom: track two-finger touch distance/center in `touchstart`/`touchmove`
  - Hide grid lines when `cellSize < 4` to avoid noise at small zoom levels
- [x] Undo / redo history stack
- [x] Export artwork as PNG
- [ ] Touch / mobile support for panning and drawing
- [ ] Flood fill (paint bucket) — press `G` to toggle, click to fill contiguous region
- [x] Eyedropper — hold `Alt` + click to pick color from an existing pixel
- [ ] Symmetry / mirror mode — toggle horizontal and/or vertical axis mirroring for sprite work
- [ ] Line tool — hold `Shift` while drawing to snap to straight lines (horizontal, vertical, 45°)
- [ ] Adjustable brush size — `[` / `]` keys to cycle 1×1, 2×2, 3×3
- [ ] Share via URL — encode small drawings into URL hash (base64 + RLE), zero-server sharing
- [ ] Onion-skin animation — multiple frames with previous frame shown as ghost overlay

## Improvements

- [ ] Handle localStorage quota errors gracefully for large drawings
- [ ] Use `addEventListener` instead of implicit global event handlers (`onresize`, `onpointerup`)
- [ ] Cache grid lines as a repeating pattern instead of redrawing every frame
- [x] Keyboard shortcuts overlay — press `?` to show translucent help panel listing all bindings
- [ ] Palette swap support — load alternate palettes (GameBoy, CGA, NES) in addition to PICO-8
