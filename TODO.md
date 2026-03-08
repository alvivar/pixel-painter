# TODO

## Features

- [x] Zoom support (scroll-zoom, pinch-to-zoom)
  - Add `scale`, `MIN_SCALE` (0.25), `MAX_SCALE` (8) state variables
  - Replace all `PX` usage with `PX * scale` in `draw()` and `cellAt()`
  - Wheel handler: zoom toward cursor (`pan = cursor - (cursor - pan) * (newScale / oldScale)`)
  - Pinch-to-zoom: track two-finger touch distance/center in `touchstart`/`touchmove`
  - Hide grid lines when `cellSize < 4` to avoid noise at small zoom levels
- [x] Undo / redo history stack
- [ ] Export artwork as PNG
- [ ] Touch / mobile support for panning and drawing

## Improvements

- [ ] Handle localStorage quota errors gracefully for large drawings
- [ ] Use `addEventListener` instead of implicit global event handlers (`onresize`, `onpointerup`)
- [ ] Cache grid lines as a repeating pattern instead of redrawing every frame
