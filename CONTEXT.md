# Measure Tool — Session Context

## Project location

```
/Users/jeppeleth/Downloads/measuretool/
  index.html          # single-file app (120 KB, 2868 lines)
  translations.json   # 44 languages × 95 keys (281 KB)
  CONTEXT.md          # this file
```

## What the project is

A self-contained browser-based image measurement tool, replicated from measureonimage.com and stripped of all ads, analytics, SEO content, and branding. All functionality runs client-side with no server. Open `index.html` directly in a browser — no build step, no dependencies except the jsPDF CDN loaded in `<head>` and pdf.js, which is loaded on demand from cdnjs only after explicit user consent when a PDF file is opened.

## Origin

Derived from measureonimage.com. The original was a single PHP-rendered HTML page. The replication:
- Removed all Google/Cloudflare ad SDKs, analytics, inView lazy-ad library
- Removed SEO/blog/FAQ content sections
- Replaced the original footer with About (jeppeleth.com) and GitHub links
- Replaced the collapsible header with a native-app menubar
- Redesigned the toolbar ribbon to match a desktop app aesthetic

## File structure — index.html

The file has no build system. Everything is inline in this order:

1. `<head>`: meta tags, jsPDF CDN script, full `<style>` block
2. `<div id="ctx-menu">`: context menu (populated dynamically by JS)
3. `<nav id="menubar">`: File | Language | About menus + "Measure Tool" brand right-aligned
4. `<div id="main-app-container">`:
   - `<div id="mobile-hint">`
   - `<select id="langSelect" style="display:none">`: hidden select kept for i18n JS sync
   - `<div id="toolbar">`: ribbon toolbar
   - `<div id="canvas-container">`: canvas, drop zone, measurement panel
   - `<div id="helpModal">`: keyboard shortcuts modal
   - `<div id="aboutModal">`: About modal (translated h1, p, badges)
   - `<div id="pdfConsentModal">`: consent dialog shown before loading pdf.js
   - `<div id="pdfPageModal">`: page picker shown for multi-page PDFs
5. Script block 1: base variable declarations, DOMContentLoaded bootstrap, base pointer/draw/export functions
6. Script block 2: overrides from block 1 with advanced features — areas, angles, measurement table, save/load project, `isHoveringHitTarget`, `useSelectedAsScale`, pointer mode, context menu IIFE, PDF support IIFE
7. i18n script block: inline English `EN` object, `applyLanguage(code)`, `fetch('./translations.json')`, `toggleHeader` (unused stub kept), `toggleMenu`, `closeMenus`, `selectLang`, `syncLangMenu`

## CSS design system

```css
--bg-color: #1e1e1e
--menubar-bg: #2d2d2d
--titlebar-bg: #252526      /* unused after titlebar removal */
--toolbar-bg: #2d2d2d
--toolbar-border: #3c3c3c
--group-bg: #383838         /* unused after flat toolbar redesign */
--group-border: #4a4a4a     /* unused after flat toolbar redesign */
--accent: #0e7aff
--success: #28cd41
--danger: #e84040
--text: #cccccc
--text-dim: #888888
--text-bright: #ffffff
--warning: #d4a017
--border: #3c3c3c
--dropdown-bg: #252526
--dropdown-hover: #3a3a3a
--dropdown-border: #555
```

## Toolbar (ribbon) design

- `#toolbar`: `display: flex; flex-direction: row; height: 58px; overflow-x: auto`
- Groups (`.group`): flat, no background/border, separated by `border-right: 1px solid var(--toolbar-border)`
- Toolbar buttons: `height: 44px`, column flex layout, icon (`span.btn-icon`, 16px) above label (`span.btn-label`, 9px dim)
- Active mode buttons: `background: rgba(14,122,255,0.12)` + `::after` accent underline at bottom
- `.menu-item` (dropdown): `height: 30px`, `padding: 3px 12px`, `font-size: 12px`
- `.ctx-item` (context menu): same as `.menu-item`
- `button.menu-trigger` (menubar): `height: 22px`, row flex, transparent background

## Toolbar group order (left to right)

1. Pointer (Pan button — `id="pointerModeBtn"`)
2. Viewport (zoom +/−/1:1/Fit)
3. History (Undo/Redo/Reset)
4. Delete (`id="deleteBtn"`, delete-btn class, warning colour)
5. Line Style & Width | Text Color & Size | Type (`id="customLineBtn"`, color pickers, selects, cap type)
6. Add Custom Text (color picker, size select, `id="addTextBtn"`)
7. Advanced / Measure (Area, Finish, Angle, separator, Table toggle `id="measurementsToggleBtn"`)
8. Scale & Units (📐 icon with title tooltip, `id="knownValue"`, `id="unitName"`)

## Menubar

Three dropdown menus, left-aligned. "Measure Tool" brand name right-aligned via `margin-left: auto`.

### File
Open image (`#imageLoader`), Save project, Load project (`#projectLoader`), separator, Export PNG/JPG/PDF

### Language
44 language items calling `selectLang(code)`, sorted alphabetically by endonym (Latin scripts A–Z, then non-Latin grouped by script). Active language highlighted with `lang-item--active` class (checkmark via CSS `::before`). Trigger label always reads "Language". The hidden `#langSelect` order is irrelevant (used only for JS value sync).

### About
- Measure Tool → opens `#aboutModal` (translated h1, description, badges)
- About the author → https://jeppeleth.com
- GitHub → https://github.com/PLACEHOLDER  ← needs updating when repo is created
- separator
- Keyboard shortcuts & Help → opens `#helpModal`

## i18n system

- `window.allTranslations`: full 44-language object loaded from `translations.json`
- `window.currentLang`: tracks active language code
- `window.translations`: shared mutable object used throughout all app JS — mutated in place by `applyLanguage()`
- English (`EN`) is inlined in the HTML so it is available immediately on parse, before the fetch completes
- `applyLanguage(code)`: mutates `translations`, sets `document.title`, `lang`, `dir` (RTL for ar/ur/fa/he), walks `[data-i18n]` and `[data-i18n-attr]` elements, calls `syncLangMenu(code)`, reloads initial SVG, calls `updateUI()`
- URL query param `?lang=<code>`: `getInitialLang()` reads it on load (validated against `SUPPORTED_LANGS`, falls back to `en`); applied after `translations.json` loads. `selectLang(code)` calls `updateLangParam(code)` which does `history.replaceState` — sets `?lang=` for non-English, removes it for English. Lets users share a URL that opens in the same language.
- `syncLangMenu(code)`: updates `.lang-item--active` class; trigger label stays "Language"
- Translation keys: 97 per language — JS object keys (svgTitle, instr1–6, showTable, hideTable, etc.) plus HTML UI strings (labelUpload, labelScaleUnits, addTextBtn, etc.) plus `labelPointer`, plus clip keys `clipBtn` (button label) and `titleClip` (tooltip), plus 9 pdf* keys (pdfConsentTitle, pdfConsentBody, pdfConsentDontAsk, pdfConsentCancel, pdfConsentContinue, pdfPageTitle, pdfPageLabel, pdfPageOpen, pdfLoadError). `svgTitle` is "Area Measure Tool" localized per language (e.g. de "Flächenmesswerkzeug", ja "面積測定ツール"). The Clip button uses `data-i18n="clipBtn"` on its label and `data-i18n-attr="title:titleClip"` on the button. `pdfPageLabel` contains a `{n}` placeholder replaced at runtime via `.replace('{n}', n)` — the first interpolated translation string in the project.

## Core data model

```js
lines[]       // array of line objects; lines[0] is ALWAYS the scale reference
areas[]       // array of polygon objects {name, points[]}
angles[]      // array of angle objects {name, vertex, armA, armB, a, b}
customTexts[] // array of {x, y, text, color, size}
```

Line object shape:
```js
{ name: '', x1, y1, x2, y2, lColor, lWidth, tColor, tSize, cap, isCustom }
```

## Scale line convention

`lines[0]` is always the scale. This is the entire mechanism:
```js
const isRef = (i === 0);
```
- isRef → colour is red `#ff3b30` (or `#ff00a2` when selected)
- isRef → label shows the known value from `#knownValue` input
- All other lines calculate length as `dist / pixelsPerUnit`
- `pixelsPerUnit = dist(lines[0]) / knownValue`
- Deleting `lines[0]` promotes `lines[1]` to scale automatically (array shift)
- "Use as scale" context menu item: `lines.splice(idx,1)[0]` then `lines.unshift(promoted)`

## Selection and hit testing

Hit radius: `HIT_RADIUS = 20` CSS pixels, scaled by `/ scale` for zoom invariance.

Priority chain in `handlePointerDown` (block 2):
1. `e.button === 2` → bail (right-click goes to context menu only)
2. 2-finger touch → pinch/zoom
3. Space/pointerMode → pan
4. areaMode → `handleAreaPoint`
5. angleMode → `handleAnglePick`
6. textMode → `createTextInput`
7. customTexts hit → select + drag text
8. areas hit (point-in-polygon) → select area
9. angles hit (label proximity) → select angle
10. lines endpoint hit → `draggedPoint`
11. lines segment hit (`distanceToSegment`) → `draggedWholeLine`
12. nothing → start new line (`isDrawing = true`)

Hover cursor: `isHoveringHitTarget(pt)` runs in `handlePointerMove` when `pointers.length === 0` (no button pressed). Returns `true` if any element is under the cursor → sets cursor to `pointer`. Only active in default draw mode (not area/angle/text/pointer modes).

## Modes

```js
pointerMode    // pan on every drag; cursor: grab/grabbing
textMode       // click to place text; cursor: text
areaMode       // click to add polygon vertices; cursor: copy
angleMode      // click two connected lines; cursor: cell
customLineMode // draws lines without measurement label
clipMode       // define export clip region; cursor: crosshair/resize/move
```

All modes are mutually exclusive. `syncModeButtons()` updates button `.active` classes and cursor after any toggle.

## Clip / export area

Toolbar button `id="clipBtn"` in the Viewport group toggles `clipMode` via `toggleClipMode()`.

State (script block 2 top):
```js
let clipMode = false;
let clipRect = null;            // { x, y, w, h } in IMAGE space; null = export whole image
let draggedClipHandle = null;   // 'nw' | 'ne' | 'sw' | 'se' | 'move'
let clipDragStart = null;       // { px, py, rect } snapshot at drag start
const CLIP_HANDLE_SIZE = 10;    // screen-px half-size of a corner handle
```

- `clipRect` lives in image space (same coordinate system as lines/areas), so it survives zoom/pan and feeds directly into export.
- Toggling clip mode on initializes `clipRect` to `defaultClipRect()` (full image) if null. Max clip area is the image/canvas size — enforced by `clampClipRect()` (min size 10px, clamped to image bounds).
- `clipHandleAt(pt)`: returns `'nw'|'ne'|'sw'|'se'` if within `CLIP_HANDLE_SIZE/scale` of a corner, `'move'` if inside the rect, else `null`.
- `updateClipDrag(pt)`: corner drags adjust the two edges meeting at that corner (normalised so w/h stay positive); `'move'` translates the whole rect. All clamped to image bounds.
- `drawClipOverlay(ctx)`: called last in `draw()`. Renders (1) a `rgba(0,0,0,0.45)` overlay outside the rect using an even-odd fill (outer image rect + inner clip rect, `fill('evenodd')`), (2) a white dashed border (`setLineDash([6/scale, 4/scale])`), (3) four blue corner handles at fixed screen size (`CLIP_HANDLE_SIZE/scale`).
- Pointer chain: in `handlePointerDown`, a `clipMode` branch sits after the pan check but before area/angle/text — clicking a handle starts a drag, clicking elsewhere does nothing. `handlePointerMove` updates the drag or sets the resize/move cursor on hover. `handlePointerUp` finalizes and re-clamps. Space-pan still works in clip mode.
- Measurements are not editable while clip mode is active (the branch returns early, like area/angle modes).
- Reset on new image load in `loadSource`: `clipRect = null; clipMode = false; draggedClipHandle = null; syncModeButtons()`.

### Export with clip
`renderExportCanvas()` computes `region` = `clipRect` only when `clipMode` is active, else the full image. Toggling the clip button off (while retaining `clipRect` for later) makes exports use the whole image again. Output canvas is sized to `region.w × region.h` (× the `OUTPUT_MAX_SIDE` fit factor `f`), and a single `tCtx.translate(-region.x, -region.y)` before `drawImage` offsets the image and all image-space measurement drawing so only the clipped region is exported. PNG/JPG/PDF all go through `renderExportCanvas` so all three honor the clip. `saveProject()` is unaffected — it always saves the whole image plus measurements.

## Snapping

`getSnappedPos(pos, currentLineIdx)`: snaps to nearest existing line endpoint within `SNAP_THRESHOLD / scale` (20px). Fires `navigator.vibrate(10)` on the snap transition (Android only; silently ignored elsewhere). Tracks `_wasSnapped` to fire only once per snap entry.

## Undo / redo

State is serialised as JSON: `{ l: lines, t: customTexts, a: areas, g: angles }`.
- `undoStack[]`: array of serialised state strings; `undoStack[0]` is the initial empty state
- `redoStack[]`: cleared on every new action
- Keyboard: Ctrl/Cmd+Z = undo, Ctrl/Cmd+Y or Ctrl/Cmd+Shift+Z = redo

## Export

- PNG / JPG: rendered by `renderExportCanvas()` to a canvas capped at `OUTPUT_MAX_SIDE = 4096` px on the longest side. The canvas is sized to `floor(w*f) × floor(h*f)` where `f = min(1, 4096 / max(w, h))`, and all drawing (image, lines, areas, angles, texts) is done under `ctx.scale(f, f)` so everything scales together. When `f = 1` behaviour is identical to before.
- PDF: uses jsPDF (`window.jspdf.jsPDF`) from CDN; orientation and page size are derived from the (already-capped) canvas produced by `renderExportCanvas()`.
- CSV: measurement table rows exported via `exportMeasurementsCsv()`
- Project JSON: `saveProject()` applies the same `f = min(1, 4096 / max(w, h))`. When `f < 1` it draws the image downscaled onto an offscreen canvas (PNG data URL) and calls `scaleMeasurements(f)` which returns deep copies of lines, areas, angles, and customTexts with all image-space fields (coordinates, `lWidth`, `tSize`, `size`) multiplied by `f`; live state is never mutated. When `f = 1` the existing data URL and live arrays are used directly. Projects loaded from a >4096-px render are stored and reload at 4096 resolution; because both the image and all coordinates are scaled by the same `f`, measurement values are invariant.

## PDF opening

Entry point is the overridden `handleFile` in the PDF IIFE (end of script block 2). A file is routed into the PDF flow when `file.type === 'application/pdf'` or the name ends with `.pdf` (case-insensitive). Non-PDF files fall through to the original `handleFile`.

If either PDF modal is already open, the incoming file is ignored and the file input is reset (re-entrancy guard).

Consent: if `localStorage.getItem('measuretool.pdfLibraryConsent') !== '1'`, `#pdfConsentModal` is shown. It has a "Don't ask again" checkbox; clicking Continue with the box ticked writes `"1"` to that key. Cancel (button, Escape, or backdrop click) calls `clearPending()` and resets the file input without loading anything. If the library was successfully imported earlier in the same page session (`pdfLibLoaded === true`), the consent modal is skipped entirely regardless of the localStorage key, because the library is already in memory and nothing new is fetched.

Library loading: `loadPdfLib()` performs a single `import()` of pdf.js from cdnjs and caches the promise so the network hit happens at most once per page load. Pinned URLs:
```
https://cdnjs.cloudflare.com/ajax/libs/pdf.js/6.3.289/pdf.min.mjs
https://cdnjs.cloudflare.com/ajax/libs/pdf.js/6.3.289/pdf.worker.min.mjs
```
`GlobalWorkerOptions.workerSrc` is set to the worker URL immediately after import. pdf.js wraps cross-origin workers in a blob internally, so this works from both `file://` and `https://` origins.

Document loading: `pdfjs.getDocument({ data: buffer })` receives the file as an `ArrayBuffer`. The returned `loadingTask` is hoisted to function scope so it can be destroyed in the catch if `task.promise` rejects (corrupt or password-protected file).

Page routing: if `pdfDoc.numPages === 1` the single page renders immediately. Otherwise `#pdfPageModal` is shown with a `<select>` listing "Page 1" … "Page N" (options built with `createElement`; label text from `translations.pdfPageLabel.replace('{n}', n)`). Cancel calls `clearPending()` which destroys the loading task. The Open button extracts the chosen page number, clears pending state, and calls `renderPdfPage`.

Rendering: `page.getViewport({ scale: 3 })` gives 3× the 72 dpi default; the longest-side cap is 8192 px on fine-pointer devices (`matchMedia('(pointer: fine)')`, guarded with `window.matchMedia &&`), and 4096 px otherwise; if the scaled result exceeds the cap the scale is reduced proportionally. An offscreen canvas is filled white before rendering so transparent PDF backgrounds appear white. The result is exported as a PNG data URL and handed to `loadSource(dataUrl, true, true)`, which is the existing block-2 override, so undo history, export, and project save/load work unchanged.

Cleanup: `clearPending()` destroys `_pendingLoadingTask` if non-null, nulls `_pendingFile`, `_pendingPdfDoc`, and `_pendingLoadingTask`, and resets the file input. Both error catches (in `proceedWithPdf` and `renderPdfPage`) call it after `loadingTask.destroy()`. On success `loadingTask.destroy()` is called inside the render `.then` before `loadSource`.

Escape handling: the PDF Escape listener is registered on `document` before the context-menu IIFE's Escape listener (source order within script block 2). When a PDF modal is open, the PDF listener calls `e.stopImmediatePropagation()` after closing it, preventing the context-menu listener from also firing. The block-1 `keydown` handler has no Escape branch and is unaffected.

Error display: uses `alert(translations.pdfLoadError)`, following the same pattern as `translations.projectLoadError`.

## Context menu

Right-click on canvas fires `contextmenu` event. Hit-tests at cursor position:
- Hit on element → shows element menu
- No hit → shows canvas menu

Element menu (line, area, angle, text):
- Delete (danger)
- If line and not index 0: separator + "📐 Use as scale" (last item)

Canvas menu:
- 🖱️ Pointer mode (with ✓ suffix when active)
- separator
- ↩ Undo (disabled when nothing to undo)
- ↪ Redo (disabled when nothing to redo)
- separator
- ⤢ Fit to window

Menu items are built as real DOM nodes via `document.createElement('button')` with `addEventListener('click', ...)` — no inline handlers, no string serialisation. Menu closes on click outside (document click listener) or Escape.

## Wheel / scroll behaviour

```js
container.addEventListener('wheel', (e) => {
    e.preventDefault();
    if (e.ctrlKey) {  // trackpad pinch or Ctrl+scroll
        zoomToPoint(e.clientX, e.clientY, factor);
    } else {          // trackpad scroll or plain mouse wheel
        offsetX -= e.deltaX;
        offsetY -= e.deltaY;
    }
});
```

## Touch / mobile

Two-finger touch: simultaneous pan + pinch-zoom via pointer events (`pointers[]` array tracking).
One-finger: draw (or pan in pointer mode).
Mobile hint bar shown below menubar on screens ≤768px.

## Known placeholder

`https://github.com/PLACEHOLDER` appears in the About menu (GitHub link) and needs updating when the repository is created.

## Things deliberately not in the app

- No analytics, tracking, or ad SDKs
- No server-side code or API calls (except `fetch('./translations.json')` and the on-demand cdnjs fetch of pdf.js after explicit user consent)
- No SEO content sections (About/How-it-works/FAQ from original site)
- No guides or blog teasers
- No language switcher that navigates to external URLs
