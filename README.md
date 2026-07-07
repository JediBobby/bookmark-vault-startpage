<div align="center">
  <img src="BVicon.svg" alt="Bookmark Vault emblem" width="180" />

  <h1>Bookmark Vault</h1>

  <p><em>A self-contained browser start page that lives in a single HTML file.</em></p>

  <p>No build step. No npm. No server. No account. Open the file — get your bookmarks.</p>
</div>

---

## What it is

Bookmark Vault is a personal start page packaged as **one self-contained HTML file**. Double-click it, set it as your browser's new-tab page, and you have a curated dashboard of collections, bookmarks, and widgets. No extension. No installer. Nothing phones home except optional favicon fetches.

Everything you add stays in your browser's `localStorage`, on this machine.

> **Now at 2.0** — a bolder redesign with themes, a command palette, a calendar, drag-and-drop, and browser-bookmark import. Data from 1.x is migrated automatically the first time you open it, and the old file keeps working. See [Upgrading from 1.x](#upgrading-from-1x).

## Features

- **Collections** — each with an icon (a curated cross-platform glyph set *or* any emoji), an accent color, and a description. Reorder by drag or arrow keys.
- **Bookmarks** — organized per collection, with:
  - **Drag-and-drop** reordering.
  - **Quick-add**: paste a URL and the title auto-fills, with a **live favicon preview** and **duplicate detection**.
  - Optional **emoji/icon override**, **tags**, and a **note** per bookmark.
  - **Favorites** pinned to a top strip (quick-launch the first nine with keys `1`–`9`).
- **Per-collection preview modes** — Chips, Grid, List, or Large (optionally with names), set globally or overridden per collection.
- **Smart search bar** — pick from 8 engines (DuckDuckGo, Google, Bing, Brave, Startpage, Kagi, Wikipedia, YouTube) and use **DuckDuckGo `!bang`** shortcuts (`!w`, `!g`, `!yt`, …) from any engine.
- **Command palette** (`Ctrl`/`Cmd`+`K`) — fuzzy-search your bookmarks **and** actions (add, import, toggle theme, jump to a collection…).
- **Widgets** — a live **clock** and time-based **greeting**, and a **calendar** (ISO week numbers, US holidays, single- or three-month view).
- **Open All** — launch every bookmark in a collection at once, with an optional confirmation, a max-tabs cap, and staggered opening to dodge popup blockers.
- **Themes** — light / dark / auto, a **custom accent color** (with automatic text-contrast), comfortable/compact **density**, and an aurora or minimal background.
- **Import / export** — import a browser's exported **bookmarks HTML** (folders become collections) or a Bookmark Vault **JSON** backup; export a versioned JSON snapshot.
- **Undo** — deleting a bookmark or collection offers a one-click undo.
- **Favicon fallback** — when a favicon can't load, a deterministic colored-letter avatar is drawn instead, at the same size so nothing shifts.
- **Accessible** — keyboard navigation, a command palette, modal focus management + trap (`inert`), ARIA labels, and `prefers-reduced-motion` support.

## Quick start

- **Live version:** open [jedibobby.github.io/claude-bookmark-startpage](https://jedibobby.github.io/claude-bookmark-startpage/) and set it as your browser's new-tab page.
- **Local copy:** download [`index.html`](index.html), double-click to open, *(optional)* set as your new-tab / home page.

That's it.

## Customizing your page

- **Top bar** — the big center field is a web search (click the engine chip to switch engines); the right cluster is **Add** (a bookmark), the **command palette**, **calendar**, a **theme** toggle, **Edit** (rearrange collections), and **Settings**.
- **Add a bookmark** — paste a URL; the title and favicon fill in automatically. Choose a collection, and optionally add an emoji, tags, or a note.
- **New collection** — the **New** button in the Collections section, or the command palette.
- **Edit / rearrange** — the **Edit** button turns collection cards into draggable, clickable tiles; drag to reorder or click a tile to edit it (icon, color, description, and a per-collection preview style).
- **Inside a collection** — add, edit, delete, favorite, and drag-reorder bookmarks; **Open all** in one click.
- **Settings** — theme, accent, density, background, default view, home widgets, week start, search engine, open-behavior, and data import/export/reset.

## Keyboard shortcuts

| Action | Keys |
|---|---|
| Command palette | `Ctrl`/`Cmd` + `K` (or `O`) |
| Focus web search | `/` |
| Quick-launch favorite 1–9 | `1` … `9` |
| Calendar | `G` then `C` |
| Close / back | `Esc` |
| Shortcut help | `?` |

## Upgrading from 1.x

Open `index.html` (2.0) and your 1.x data appears automatically:

- **Same browser (automatic):** 1.x kept state in four keys (`startpage:groups`, `startpage:bookmarks`, `startpage:settings`, `startpage:view`). 2.0 reads them **once**, migrates everything into a single key (`bookmarkvault:v2`), and **leaves the old keys untouched** — so the previous version still opens with its data intact.
- **Different browser or machine (via file):** in 1.x use **Settings → Export**, then in 2.0 use **Settings → Import JSON** — 2.0 auto-detects the 1.x export format and converts it. (No need to touch the old file.)
- Either way, groups become collections; bookmarks, favorites, custom glyphs, and your settings all carry over.
- The previous single-file build is preserved in this repo as [`index-v1.html`](index-v1.html) (safe to delete once you're happy with 2.0).

## Project layout

```
index.html      — The whole app (v2): HTML + CSS + JS, all inlined
index-v1.html   — The previous 1.x build, kept for reference (safe to delete)
BVicon.svg      — Design source for the emblem (the runtime copy is inlined in index.html)
README.md
LICENSE
.nojekyll       — Tells GitHub Pages to skip Jekyll processing
```

## Technical notes

- **Single self-contained file.** Open via `file://` or serve over HTTP — both work.
- **No build, no bundler, no npm.** Edit the file, refresh the page.
- **Fonts** pair an elegant system serif for display (Cormorant / Hoefler / Didot / Palatino … → Georgia) with a native UI sans (`system-ui`, Segoe UI, Roboto …). Nothing is loaded over the network.
- **State** lives in `localStorage` under one key: `bookmarkvault:v2` (collections, bookmarks, notes, settings). Legacy 1.x keys are read for migration and then left alone.
- **Favicons** are fetched from DuckDuckGo (`https://icons.duckduckgo.com/ip3/<domain>.ico`). The colored-letter avatar fires on actual network failures (offline / DNS / blocked).
- **Browser support.** Modern Firefox, Chrome, Brave, Edge, and Safari. Uses modern CSS (`color-mix`, `inert`, container/`data-*` theming); older browsers miss the polish but core functionality works.

## Data & privacy

- All your data stays in this browser, on this machine.
- No analytics, no telemetry, no remote sync.
- The only outbound network call is the favicon request to DuckDuckGo, one per visible bookmark. Web search runs only when you submit a query, to the engine you chose.
- Use **Settings → Data → Export** to download a JSON snapshot at any time.

## Import formats

**Browser bookmarks (HTML).** Export bookmarks from any major browser and import the resulting Netscape HTML file — top-level folders become collections and their links become bookmarks.

**Bookmark Vault JSON.**

```json
{
  "version": 2,
  "collections": [
    { "id": "1", "name": "My Collection", "description": "...",
      "icon": "◆", "accentColor": "#c9a227" }
  ],
  "bookmarks": [
    { "id": "b1", "title": "Example", "url": "https://example.com/",
      "isFavorite": false, "collectionId": "1",
      "tags": [], "note": "", "customIcon": "" }
  ],
  "notes": [],
  "settings": {}
}
```

Imports are validated: each collection needs an `id` and a `name`; each bookmark needs a `url` and a `collectionId` that matches an imported collection. Invalid items are skipped.

The importer also accepts a **1.x export** (`{ groups, bookmarks, … }`) and converts it automatically — so an export from the old version imports directly.

## License

[MIT](LICENSE) — do whatever you want, just keep the notice.
