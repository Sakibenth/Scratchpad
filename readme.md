# PageNotes — Sketchpad Reader

A split-pane study tool that puts a PDF, a YouTube video, or a local video file on one side and a freeform sketchpad on the other — so you can read, watch, and take notes at the same time, without switching apps.

Everything runs in a single HTML file, entirely in your browser. Nothing is uploaded anywhere; your PDFs, videos, and notes never leave your machine.

---

## Features

### Three content sources

- **PDF reader** — Open any PDF, scroll or jump between pages, zoom in/out or fit to width. Pages render lazily as you scroll, so even large PDFs open quickly.
- **YouTube embed** — Paste a YouTube link (full URLs, `youtu.be` short links, `shorts/`, `embed/`, or just the bare video ID all work) and watch it inline.
- **Local video playback** — Open or drag in a video file from your computer and play it with a native browser video player.

Only one source is active at a time. Switching sources cleanly stops whatever was playing before (so audio doesn't keep running in the background) and tears down the previous view.

Click the file name in the header at any time to rename how it's displayed — useful for giving a YouTube video or a vaguely-named PDF a clearer label. This only changes the display name; it never affects how or where your notes are actually stored, so renaming is always safe and won't disconnect you from previously saved notes (local or on Drive).

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

### Google Drive sync (optional, manual)

Once set up (see [Google Drive setup](#google-drive-setup-optional) below), a **☁ Connect Drive** button appears in the header. After connecting your Google account, you get two more buttons:

- **⬆ Save to Drive** — asks you what to name the file (defaulting to its previous name, or an auto-generated one the first time), then bundles all your sketch pages for the currently open PDF/video and uploads it to your own Google Drive. Saving again later overwrites that same file rather than creating duplicates — even if you give it a different name each time, since the app remembers which Drive file belongs to which source internally.
- **⬇ Load from Drive** — finds that file (by the link it remembers, falling back to a name search if needed) and restores it into this browser's local storage, overwriting whatever's currently there for this file (you'll be asked to confirm first, and shown the file's current name).

This is entirely manual — nothing uploads or downloads automatically. It exists specifically to get your notes out of one browser's local storage so they can follow you to a different device or survive a cleared cache, without needing any server of your own.

Only the JSON note data is stored on Drive — never your PDF, video file, or YouTube link. The app uses Google's `drive.file` scope, the narrowest permission Google offers: it can only see and modify files this app itself created, never anything else in your Drive. Renaming the file — whether through the app's save prompt or directly in Drive's own interface — never breaks the app's ability to find it again later.

### Export to PDF

The **Export Notes PDF** button asks what to name the downloaded file (pre-filled with a sensible default), then bundles every sketchpad page that has actual content into a single PDF — one page per sketch page, labeled "Page N." Empty pages are automatically skipped. This is the only way to get your notes out of the browser permanently.

Strokes are exported as real vector paths rather than flattened images, so files stay small even for pages that are mostly blank with just a few notes. If a page's strokes can't be vectorized for some reason, that page automatically falls back to a rasterized image instead of breaking the export — you'll see a note in the success toast if this happens.

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

## Google Drive setup (optional)

Saving and loading notes to Google Drive needs your own free Google API credentials. This is unavoidable for a single-file app with no backend server — there's no shared key that could be safely shipped inside the HTML, since anyone reading the page source would see it. Registering your own costs nothing and takes about 5 minutes.

1. Go to **[console.cloud.google.com/projectcreate](https://console.cloud.google.com/projectcreate)** and create a new project (any name is fine).
2. In that project, go to **APIs & Services → Enabled APIs & services → + ENABLE APIS AND SERVICES**, search for **"Google Drive API,"** and enable it.
3. Go to **APIs & Services → OAuth consent screen**. Choose **External**, fill in an app name and your email, and save. You can leave the app in **"Testing"** mode — just add your own Google account under **"Test users"** so you're allowed to sign in.
4. Go to **APIs & Services → Credentials → + CREATE CREDENTIALS → OAuth client ID**. For application type, choose **"Web application."**
5. Under **"Authorized JavaScript origins,"** add the exact URL you'll open this app from — for example `http://localhost:8000` (no trailing slash, no path after it). If you later run it from a different origin, add that too.
6. Click **Create**. Copy the generated **Client ID** (it looks like `123456789-abc123.apps.googleusercontent.com`).
7. Open `PageNotes.html` in a text editor, find this line near the top of the `<script>` block:

   ```js
   const GOOGLE_DRIVE_CLIENT_ID = 'YOUR_GOOGLE_CLIENT_ID_HERE.apps.googleusercontent.com';
   ```

   and replace the placeholder with your real Client ID.
8. Save the file and reload it from the same origin you authorized in step 5. Click **☁ Connect Drive** in the header — Google's sign-in popup should appear.

**A few things worth knowing:**

- Because the app is in "Testing" mode, only the test users you listed in step 3 can sign in. This is the right setting for personal use; you don't need to submit the app for Google's verification review unless you intend for *other* people to use your specific Client ID.
- The `drive.file` scope means this app can only ever see files it created itself — not your existing Drive contents. There's nothing to configure for this; it's the narrowest scope Google offers and is hardcoded into the app.
- If you ever want to revoke access, visit [myaccount.google.com/permissions](https://myaccount.google.com/permissions) and remove the app from there.

---

## File structure

This is intentionally a single self-contained HTML file — there's nothing to install and no build step. All you need is:

```
your-folder/
└── PageNotes.html
```

External libraries (PDF.js, Fabric.js, jsPDF, svg2pdf.js, Google Identity Services) are loaded from a CDN at runtime, so you'll need an internet connection the first time the page loads, even though your actual files stay local.

---

## Limitations & known issues

- **Notes are stored per-browser, per-device by default.** They're saved under a key derived from the file's name, size, and last-modified date. Reopening a *different copy* of the same PDF (re-downloaded, renamed, or edited) won't match, and notes won't carry over automatically. **Google Drive sync (see above) is the way around this** — set it up once and you can move notes between browsers/devices manually.
- **Storage has a size ceiling.** Browsers cap `localStorage` at roughly 5–10MB per site. Heavy sketching across many pages can hit this; the app will warn you with a toast if a save fails. Exporting to PDF or saving to Drive periodically sidesteps this entirely.
- **Clearing browser data wipes notes.** Clearing your browser's site data/cache for this page, switching browsers, or switching devices all mean your saved notes won't be there unless you'd already saved a copy to Drive or exported to PDF first.
- **Drive sync requires your own free API credentials and a real HTTP origin** (not `file://`) — see [Google Drive setup](#google-drive-setup-optional) above. Access tokens expire after about an hour; if a save/load fails with a "session expired" message, just click **Connect Drive** again.
- **YouTube needs a real HTTP server**, as described above — this is a restriction from YouTube's player, not something the app can work around when run from local disk.
- **No PDF-page or video-timestamp linking.** Sketch pages are a separate notebook by design — there's no automatic association between sketch page 3 and PDF page 3, or between a note and the video's playback position at the time you wrote it.

---

## Future Improvements

Roughly in order of likely usefulness:

- **Optional page linking** — an opt-in way to tag a sketch page with "this goes with PDF page N" or "this goes with video timestamp 14:32," without forcing a rigid 1:1 mapping.
- **Dropbox as a second sync option** — same manual save/load model as Google Drive, for people who prefer Dropbox.
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