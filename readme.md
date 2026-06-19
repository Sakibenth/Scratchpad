# PageNotes — Scratchpad Reader

A split-pane study tool that puts a PDF, a YouTube video, or a local video file on one side and a freeform sketchpad on the other — so you can read, watch, and take notes at the same time, without switching apps.

Everything runs in a single HTML file, entirely in your browser. Nothing is uploaded anywhere; your PDFs, videos, and notes never leave your machine.

---

## Features

### Three content sources

- **PDF reader** — Open any PDF, scroll or jump between pages, zoom in/out or fit to width. Pages render lazily as you scroll, so even large PDFs open quickly.
- **YouTube embed** — Paste a YouTube link (full URLs, `youtu.be` short links, `shorts/`, `embed/`, or just the bare video ID all work) and watch it inline.
- **Local video playback** — Open or drag in a video file from your computer and play it with a native browser video player.

Only one source is active at a time. Switching sources cleanly stops whatever was playing before (so audio doesn't keep running in the background) and tears down the previous view.

### Independent sketchpad notebook

The sketchpad on the right is its own notebook — it is **not** tied to PDF page numbers or video timestamps. It starts with one blank page, and you add more pages yourself:

- **Pen tool** with adjustable color and stroke width
- **Highlighter** for translucent marker-style strokes
- **Object eraser** — click or drag over a stroke to remove it
- **Select tool** — move, resize, copy, paste, duplicate, and group/ungroup your strokes
- **Page styles** — blank, lined, or grid background, plus a custom background color
- **Pan and zoom** on the canvas independently of the source pane (scroll to pan, Ctrl/Cmd + scroll to zoom, or use the on-screen pan buttons)
- **Multiple pages** — use **+ Page** to add a new blank page, navigate with the prev/next arrows, and delete the last page if you added one by mistake (only the last page can be deleted, to avoid silently shifting every other page's saved data)

### Autosave, always

Your strokes save to the browser's local storage continuously — after every stroke, every page switch, every edit. Closing the browser tab does **not** erase your notes. Reopening the same file later (same browser, same device, same source file) restores exactly where you left off.

This is a convenience save, not a backup. See [Limitations](#limitations--known-issues) below for when it can let you down.

### Export to PDF

The **Export Notes PDF** button bundles every sketchpad page that has actual content into a single downloadable PDF — one page per sketch page, labeled "Page N." Empty pages are automatically skipped. This is the only way to get your notes out of the browser permanently.

---

## How to run this

You can open `PageNotes.html` directly by double-clicking it, and the PDF reader and sketchpad will work fine that way.

**However, the YouTube embed will not work if you open the file directly** (i.e., as a `file:///...` URL). YouTube's embedded player requires a valid referrer/origin, and browsers send none when loading a page from local disk. You'll see YouTube's **"Error 153: Video player configuration error"** if you try this.

To fix it, serve the folder over a local web server instead, so the page loads as `http://localhost:...` rather than `file:///...`.

### Option 1 — Python (most machines already have this)

```bash
cd path/to/the/folder
python3 -m http.server 8000
```

Then open **http://localhost:8000/PageNotes.html** in your browser.

(On some Windows installs the command is `python` instead of `python3`.)

### Option 2 — Node.js

```bash
cd path/to/the/folder
npx http-server -p 8000
```

Then open **http://localhost:8000/PageNotes.html**.

### Option 3 — VS Code "Live Server" extension

Install the **Live Server** extension, right-click `PageNotes.html` in the file explorer, and choose **"Open with Live Server."** This starts a local server and opens the page for you automatically.

> Local PDF viewing and local video playback work fine either way — the server is only required for the YouTube embed feature.

---

## File structure

This is intentionally a single self-contained HTML file — there's nothing to install and no build step. All you need is:

```
your-folder/
└── PageNotes.html
```

External libraries (PDF.js, Fabric.js, jsPDF) are loaded from a CDN at runtime, so you'll need an internet connection the first time the page loads, even though your actual files stay local.

---

## Limitations & known issues

- **Notes are stored per-browser, per-device.** They're saved under a key derived from the file's name, size, and last-modified date. Reopening a *different copy* of the same PDF (re-downloaded, renamed, or edited) won't match, and notes won't carry over automatically.
- **Storage has a size ceiling.** Browsers cap `localStorage` at roughly 5–10MB per site. Heavy sketching across many pages can hit this; the app will warn you with a toast if a save fails, but it's worth exporting periodically if you're taking a lot of notes.
- **Clearing browser data wipes notes.** Clearing your browser's site data/cache for this page, switching browsers, or switching devices all mean your saved notes won't be there — export first if that matters to you.
- **YouTube needs a real HTTP server**, as described above — this is a restriction from YouTube's player, not something the app can work around when run from local disk.
- **Exported pages are rasterized images, not vector strokes.** Even simple sketches get embedded as PNGs inside the PDF, so file sizes per page are larger than the stroke data would suggest. See [Future Improvements](#future-improvements) for the planned fix.
- **No PDF-page or video-timestamp linking.** Sketch pages are a separate notebook by design — there's no automatic association between sketch page 3 and PDF page 3, or between a note and the video's playback position at the time you wrote it.

---

## Future Improvements

Roughly in order of likely usefulness:

- **Vector export** — embed actual stroke paths (via SVG) into the exported PDF instead of rasterized PNGs, dramatically shrinking file size for mostly-blank pages.
- **Optional page linking** — an opt-in way to tag a sketch page with "this goes with PDF page N" or "this goes with video timestamp 14:32," without forcing a rigid 1:1 mapping.
- **Cloud/file-based persistence** — save notes as a portable `.json` or sidecar file next to the source PDF/video, instead of relying solely on browser local storage, so notes can move between devices and survive a cleared cache.
- **Local video timestamp scrubbing tied to notes** — jump back to the moment in a lecture video where a particular note was written.
- **Text annotation tool** — typed text boxes on the sketchpad, not just freehand strokes.
- **Undo/redo** for sketchpad actions.
- **Bulk page management** — reorder pages, duplicate a page, or delete pages from the middle safely (currently only the last page can be deleted, to avoid corrupting storage keys for everything after it).
- **Multi-file session view** — keep more than one PDF/video + notebook open in tabs within the app, rather than one source at a time.

---

## Quick reference: keyboard shortcuts

| Key | Action |
|---|---|
| `V` | Select tool |
| `P` | Pen tool |
| `H` | Highlighter |
| `E` | Object eraser |
| `Delete` / `Backspace` | Delete selected object(s) |
| `Ctrl/Cmd + C` | Copy |
| `Ctrl/Cmd + V` | Paste |
| `Ctrl/Cmd + D` | Duplicate |
| `Ctrl/Cmd + G` | Group / ungroup selection |
| `Alt + drag` or middle-click drag | Pan the sketchpad canvas |
| `Ctrl/Cmd + scroll` | Zoom the sketchpad canvas |