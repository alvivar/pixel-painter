# Code Review — Simplicity & Performance

Reviewing `index.html` (~290 lines) as of current state, with zoom + center-on-last-pixel features.

## Performance

### 1. Draw loop parses every cell key as a string — every frame

**Lines 95-101.** The filled-cell loop iterates the entire `Map`, calling `indexOf` + `slice` + unary `+` on every `"x,y"` key, even for off-screen cells. For a drawing with 10k+ cells this string parsing dominates frame time.

**Fix:** Store cells with numeric coordinates (e.g. a `Map<string, {x, y, ci}>` or parallel typed arrays) so the draw loop doesn't parse strings. Alternatively, use a single integer key like `((x + 0x8000) << 16) | (y & 0xFFFF)` and decode with bit shifts.

### 2. Full canvas clear + redraw on every dirty frame

The entire canvas — background, grid, all cells, palette bar — is redrawn whenever `dirty` is set. Panning, zooming, and single-pixel paints all trigger the same full redraw.

**Fix (low priority):** The palette bar only changes on color selection — it could be drawn to an offscreen canvas and `drawImage`'d in. Grid lines at a given scale could be cached as a `CanvasPattern`. These are micro-optimizations that only matter with very large cell counts.

### 3. `[...filled]` in `save()` copies the entire Map

**Line 144.** Every save (debounced to 300ms) spreads the Map into a fresh array, then JSON-stringifies it. For large drawings this is a double allocation.

**Fix:** Build JSON incrementally, or accept the cost since saves are infrequent and debounced. Not urgent.

### 4. `PX * scale` recomputed in multiple places

Calculated in `draw()`, `cellAt()`, and the centering block. Trivial cost, but a cached `cellSize` updated when `scale` changes would be cleaner.

## Simplicity

### 5. Zoom clamp is repeated 3 times

The pattern `Math.min(MAX_SCALE, Math.max(MIN_SCALE, scale * factor))` appears in the wheel handler, touchstart zoom, and touchmove zoom.

**Fix:** Extract a helper:

```js
function clampScale(s) {
  return Math.min(MAX_SCALE, Math.max(MIN_SCALE, s));
}
```

### 6. `"x,y"` key parsing duplicated

The `indexOf(",")` / `slice` pattern for parsing cell keys appears in `draw()` (line 96) and the centering block (line 307). `cellAt()` builds the same format.

**Fix:** A pair of helpers (`parseKey(k)` → `[x,y]`, `makeKey(x,y)` → string) would centralize the format. Or switch to numeric keys entirely (see #1).

### 7. Global event handlers (`onresize`, `onpointerup`)

**Lines 210, 302.** Setting `onpointerup` and `onresize` as implicit globals is fragile — any other script assigning to them would break the app. Already noted in TODO.

**Fix:** `window.addEventListener("resize", resize)` and `window.addEventListener("pointerup", ...)`.

## Correctness

### 8. Shift+scroll lost vertical panning

**Line 222.** `panX -= e.deltaX || e.deltaY` only pans the X axis. The old wheel handler panned both axes (`panX -= e.deltaX; panY -= e.deltaY`). Vertical scroll-to-pan is now impossible via scroll wheel — only middle-click drag pans vertically.

**Fix:** Either pan both axes under shift (`panX -= e.deltaX; panY -= e.deltaY`), or drop the shift modifier entirely and use Ctrl+scroll for zoom (matching browser/Figma convention), leaving unmodified scroll for panning.

### 9. Pinch-to-zoom may conflict with pointer events

Two-finger touch fires both `touchstart`/`touchmove` (for pinch zoom) and `pointerdown`/`pointermove` (which may paint or erase). There's no guard to suppress painting during a pinch gesture.

**Fix:** Set `touch-action: none` in CSS on the canvas and track active touch count. Skip `applyPaint` when two or more touches are active. Or consolidate everything under pointer events using `e.pointerType === "touch"` and a touch-count tracker.

### 10. Negative modulo on grid offset

**Lines 75-76.** `panX % S` produces a negative remainder in JS when `panX` is negative. The grid loop `for (let x = ox; ...)` still works — it just starts one cell off-screen, drawing one wasted line per axis.

**Fix:** `((panX % S) + S) % S` guarantees a positive offset. Trivial cost savings.

## Summary

| #   | Category    | Severity        | Effort |
| --- | ----------- | --------------- | ------ |
| 1   | Perf        | High (at scale) | Medium |
| 2   | Perf        | Low             | Medium |
| 3   | Perf        | Low             | Low    |
| 4   | Simplicity  | Low             | Low    |
| 5   | Simplicity  | Low             | Low    |
| 6   | Simplicity  | Low             | Low    |
| 7   | Simplicity  | Low             | Low    |
| 8   | Correctness | Medium          | Low    |
| 9   | Correctness | Medium          | Medium |
| 10  | Correctness | Low             | Low    |

## Fix Batches

### ~~Batch A — Correctness bugs (#8, #9, #10)~~ ✅ Done

- ~~**#8** Shift+scroll panning: restore both axes (`panX -= e.deltaX; panY -= e.deltaY`)~~ ✅
- ~~**#9** Pinch/pointer conflict: add `touch-action: none` CSS, track `activeTouches` count, suppress `applyPaint` when ≥ 2 touches~~ ✅
- ~~**#10** Negative modulo: `((panX % S) + S) % S` in draw~~ ✅

### ~~Batch B — Key format & data access (#1, #4, #6)~~ ✅ Done

- ~~**#1** `filled` now stores `{ x, y, ci }` objects — draw loop reads coords directly, zero string parsing per frame~~ ✅
- ~~**#6** Centralized key format into `makeKey(x,y)` / `parseKey(k)` helpers; `cellAt()` returns `{ key, x, y }`~~ ✅
- ~~**#4** Cached `cellSize = PX * scale`, updated in wheel and pinch handlers; `draw()`, `cellAt()`, and centering all read `cellSize`~~ ✅
- Save format unchanged (`[key, ci]` pairs) — fully backward-compatible with existing localStorage data
- `lastModified` string replaced with numeric `lastModifiedX` / `lastModifiedY` — centering block no longer parses

### ~~Batch C — Event handler cleanup (#5, #7)~~ ✅ Done

- ~~**#5** Extracted `clampScale(s)` helper, replaced 2 inline clamp sites (wheel and pinch-to-zoom handlers)~~ ✅
- ~~**#7** Replaced `onpointerup` and `onresize` globals with `window.addEventListener`~~ ✅

### ~~Batch D — Rendering optimization (#2, #3)~~ ✅ Done

- ~~**#2** Palette bar rendered to offscreen canvas, redrawn only on color change or resize, blitted via `drawImage`~~ ✅
- Grid `CanvasPattern` skipped — fractional `cellSize` (e.g. `24 × 0.33 = 7.92`) causes visible drift between pattern grid lines and actual cell positions after many repetitions; the grid loop is already O(screen) not O(cells) so there's no scaling concern
- ~~**#3** `save()` builds JSON string directly — no intermediate array, no `JSON.stringify`; output format unchanged for backward compatibility~~ ✅
