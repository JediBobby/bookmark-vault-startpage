# Bookmark Vault — Improvement Roadmap

Audit date: 2026-05-25
Scope: single-file HTML app (`start-page.html`, 2911 lines)

---

## 1. `javascript:` URL Injection via Bookmark Form

- **Category:** security
- **Impact: 5 / Effort: 1 / Confidence: H**
- **Evidence:** `start-page.html:2635` — `saveBookmark()` only checks `!f.url.trim()`. The saved URL is rendered into `href` attributes at lines 2026, 2106, 2137, 2477. A user (or anyone with localStorage access) can store `javascript:alert(document.cookie)` and it executes on click despite HTML-attribute escaping.
- **Why it matters:** This is the only user-controlled input that flows into an executable sink. Any `javascript:` or `data:` URL stored as a bookmark becomes a persistent XSS vector.
- **Success criterion:** Bookmark save rejects URLs whose scheme is not `http:` or `https:`. Existing bookmarks with other schemes are rendered inert (e.g., `href="#"` with a warning).
- **Suggested approach:** Add a URL-scheme allowlist check in `saveBookmark()` and a sanitizing wrapper around `escapeAttr(b.url)` where it appears in `href` attributes.

---

## 2. "Open All" Blocked by Popup Blockers

- **Category:** correctness
- **Impact: 4 / Effort: 3 / Confidence: H**
- **Evidence:** `start-page.html:2690-2700` — `openAllBookmarks()` calls `window.open(b.url, '_blank')` in a `forEach` loop. Modern browsers block all but the first `window.open` call that isn't a direct result of a single user gesture.
- **Why it matters:** The headline feature of the app — opening an entire collection at once — silently fails for most users. Only 1 of N tabs actually opens.
- **Success criterion:** All bookmarks up to `maxTabsLimit` reliably open in separate tabs in Chrome, Firefox, and Edge without popup-blocker intervention.
- **Suggested approach:** Sequential `window.open` with brief delays and a user-facing warning about popup-blocker settings.

---

## 3. No Data Export / Import

- **Category:** UX
- **Impact: 4 / Effort: 2 / Confidence: H**
- **Evidence:** `start-page.html:1762-1788` — all state lives in localStorage under four keys. There is no UI or code path for exporting or importing this data. The settings view (`renderSettings()`, line 2251) has no export/import controls.
- **Why it matters:** A cleared browser profile, a different machine, or a localStorage quota error permanently loses all bookmarks. Users have no way to back up or migrate.
- **Success criterion:** Settings view offers "Export JSON" (downloads a file) and "Import JSON" (file picker with validation) that round-trip all groups, bookmarks, and settings.
- **Suggested approach:** Add two buttons to the settings card that serialize/deserialize the four localStorage keys as a single JSON file via `Blob`/`URL.createObjectURL` and a hidden `<input type="file">`.

---

## 4. No Version Control Initialized

- **Category:** DX
- **Impact: 4 / Effort: 1 / Confidence: H**
- **Evidence:** System context reports `Is a git repository: false`. The project directory contains only `start-page.html`.
- **Why it matters:** No history, no branches, no ability to revert changes. Every edit is irreversible and untracked.
- **Success criterion:** Project is a git repo with an initial commit and a `.gitignore`.
- **Suggested approach:** Run `git init`, add a `.gitignore`, and commit the current state.

---

## 5. No Test Coverage

- **Category:** tests
- **Impact: 4 / Effort: 4 / Confidence: H**
- **Evidence:** The project contains a single HTML file with no test files, no test runner configuration, no `package.json`. Functions like `escapeHtml`, `generateId`, `getAbbreviatedBookmarkTitle`, `bookmarksInGroup`, state mutation logic, and URL validation are all untested.
- **Why it matters:** The app has ~30 functions handling state, rendering, and user interaction with zero automated verification. Regressions will be caught only by manual testing.
- **Success criterion:** Core logic functions (`escapeHtml`, `generateId`, `getAbbreviatedBookmarkTitle`, `saveBookmark` validation, `persistAll`/`loadJSON` round-trip) have unit tests with >80% line coverage of extracted logic.
- **Suggested approach:** Extract pure functions into a separate JS module, add a lightweight test runner (e.g., Vitest), and write unit tests for data-manipulation and validation functions.

---

## 6. Space Key Hijacks Native Button Activation

- **Category:** correctness
- **Impact: 3 / Effort: 1 / Confidence: H**
- **Evidence:** `start-page.html:2848-2854` — the `handleKeydown` handler calls `e.preventDefault()` on any Space keypress when the active element is not an INPUT or TEXTAREA. This prevents native spacebar activation of focused `<button>` elements (e.g., nav actions, toggle buttons, bookmark action buttons) and prevents page scrolling.
- **Why it matters:** Keyboard-only users cannot activate buttons with Space, breaking standard accessibility expectations. Floating mode activates unexpectedly during normal keyboard navigation.
- **Success criterion:** Space key activates focused buttons per native behavior; floating mode toggles only via a dedicated keybinding (e.g., a modifier combo) or only when no interactive element is focused.
- **Suggested approach:** Check `document.activeElement` before intercepting Space, and skip the floating-mode toggle if focus is on any interactive element.

---

## 7. No localStorage Schema Validation

- **Category:** correctness
- **Impact: 3 / Effort: 2 / Confidence: H**
- **Evidence:** `start-page.html:1769-1777` — `loadJSON()` does `JSON.parse(raw)` and returns the result directly. No validation that the parsed object matches the expected shape (e.g., that groups have `id`, `name`, `accentColor`; that bookmarks have `url`, `groupId`). Corrupted or hand-edited localStorage causes silent failures or crashes.
- **Why it matters:** Users who modify localStorage (common with power users and dev tools), or whose storage gets partially corrupted, see a broken app with no diagnostics.
- **Success criterion:** `loadJSON` validates loaded data against expected shapes and falls back to defaults for individual malformed entries rather than the entire dataset.
- **Suggested approach:** Add lightweight runtime validation (check required fields exist and have correct types) in `loadJSON` with per-entry fallback.

---

## 8. Hardcoded Session Tokens in Default Bookmark URLs

- **Category:** security
- **Impact: 3 / Effort: 1 / Confidence: H**
- **Evidence:** Multiple default bookmark URLs contain authentication parameters: line 1684 (Gmail with `sacu`, `rip`, `flowName`), line 1685 (Outlook with `rpsnv`, `ct`, `rver`, `CBCXT` session params), line 1706 (Apple Store with `ssi=1AAABhNs...` token), line 1675 (Coinbase with `login_challenge=9ea5faf...`), line 1733 (ChatGPT with `error=OAuthCallback`).
- **Why it matters:** While these tokens are almost certainly expired, embedding session/auth parameters in source code is a bad practice that leaks URL structure and authentication flow details. If this file is shared or version-controlled publicly, it exposes personal service usage patterns.
- **Success criterion:** All default bookmark URLs use clean base URLs without query-string tokens or session parameters.
- **Suggested approach:** Strip query parameters from default URLs, keeping only the base domain paths needed to reach each service's login or home page.

---

## 9. Single Monolithic File

- **Category:** maintainability
- **Impact: 3 / Effort: 4 / Confidence: H**
- **Evidence:** `start-page.html` is 2,911 lines containing ~1,566 lines of CSS, ~155 lines of default data, and ~1,190 lines of JS. All logic, styling, data, and markup are in a single file with no module boundaries.
- **Why it matters:** The "no build step" constraint is a stated design goal (line 4), so splitting into separate files has a real tradeoff. However, the current size makes navigation, diffing, and targeted editing difficult. CSS changes require scrolling past 1,500 lines to reach JS.
- **Success criterion:** Proxy: CSS, JS, and default data are logically separated (either via `<style>`/`<script>` tag boundaries with clear section markers, or via separate files loaded with `<link>`/`<script src>`).
- **Suggested approach:** If the single-file constraint must remain, add prominent section-separator comments and a table of contents at the top; otherwise, split into `style.css`, `app.js`, and `data.js` loaded from the same directory.

---

## 10. Full DOM Rebuild on Every State Change

- **Category:** performance
- **Impact: 3 / Effort: 4 / Confidence: M**
- **Evidence:** `start-page.html:1907-1929` — `render()` sets `root.innerHTML` to the full app HTML on every state change (view toggle, favorite toggle, modal open/close, etc.). This destroys all DOM state: scroll positions, focus, CSS transitions mid-flight, and image load state (causing favicon re-fetches).
- **Why it matters:** With 121 bookmarks each loading a favicon image, every re-render triggers ~121 network requests to Google's favicon API. Focus loss after clicking action buttons degrades keyboard navigation. The special case at line 2721 (`updateGroupFieldFromInput`) shows the pain is already recognized.
- **Success criterion:** Proxy: favicon images do not re-request on state changes that don't affect the bookmark list; focus is preserved after toggling a favorite.
- **Suggested approach:** Move to targeted DOM updates (similar to the existing `updateGroupFieldFromInput` pattern) for high-frequency operations, or adopt a lightweight virtual-DOM library.

---

## 11. No Bookmark Search or Filter

- **Category:** UX
- **Impact: 3 / Effort: 2 / Confidence: M**
- **Evidence:** With 121 default bookmarks across 10 groups, there is no search input or filter mechanism anywhere in the UI. The only way to find a bookmark is to visually scan group cards or open each group detail view.
- **Why it matters:** As the bookmark count grows, locating a specific bookmark becomes increasingly tedious. The app's value proposition depends on quick access to bookmarks.
- **Success criterion:** A search input in the header filters bookmarks across all groups by title substring match, showing results as a flat list with group labels.
- **Suggested approach:** Add a search input to the header that filters `state.bookmarks` by case-insensitive title match and renders a temporary flat results view.

---

## 12. External Google Fonts Loaded Without Subresource Integrity

- **Category:** security
- **Impact: 2 / Effort: 1 / Confidence: H**
- **Evidence:** `start-page.html:26` — `<link href="https://fonts.googleapis.com/css2?family=...">` loads external CSS without an `integrity` attribute. The app does provide a serif fallback (line 32), so the page is functional without the fonts.
- **Why it matters:** A compromised CDN or MITM on the font CSS request could inject arbitrary CSS (e.g., exfiltration via `url()` in font-face, visual spoofing). Risk is low given Google's infrastructure but non-zero for a page used as a browser start page.
- **Success criterion:** Font CSS is either self-hosted or loaded with an SRI hash, or the external load is documented as an accepted risk.
- **Suggested approach:** Self-host the two font families by downloading the WOFF2 files and inlining the `@font-face` declarations (or base64-encoding for single-file constraint).

---

## 13. Favicon Images Not Lazy-Loaded

- **Category:** performance
- **Impact: 2 / Effort: 1 / Confidence: H**
- **Evidence:** `start-page.html:2030, 2110` — `<img src="${escapeAttr(getFaviconUrl(b.url))}" ...>` renders all favicon images eagerly. On the home view with 121 bookmarks visible in group cards, this triggers ~121 simultaneous HTTP requests to `www.google.com/s2/favicons`.
- **Why it matters:** Initial page load fires 121+ network requests for 16x16/32x32 images, many of which are below the fold. This slows first paint and wastes bandwidth.
- **Success criterion:** Favicon images below the fold load only when scrolled into view. Initial page load makes fewer than 40 image requests.
- **Suggested approach:** Add `loading="lazy"` to all favicon `<img>` tags.

---

## 14. Clock Timer Drifts From Actual Time

- **Category:** correctness
- **Impact: 1 / Effort: 1 / Confidence: H**
- **Evidence:** `start-page.html:2893` — `setInterval(updateDateTime, 60_000)` fires every 60 seconds from the moment `init()` runs, not aligned to the minute boundary. If `init()` runs at 10:30:45, the clock shows "10:30" for 60 seconds (until 10:31:45), then shows "10:31" — always lagging up to 59 seconds behind.
- **Why it matters:** As a browser start page with a prominent clock display, time accuracy matters for user trust. The drift compounds subtly and is never self-correcting.
- **Success criterion:** Displayed time is always within 1 second of the actual system time.
- **Suggested approach:** After each `updateDateTime` call, compute the milliseconds remaining until the next minute boundary and use `setTimeout` to schedule the next update at that exact moment.

---

## Themes

The project demonstrates thoughtful UI polish and careful HTML escaping, but its single-file architecture has reached a scale where maintainability and testability are significantly impaired. Security findings cluster around input validation boundaries — the app correctly escapes output but does not validate input schemas (URLs, localStorage data). The full-DOM-rebuild rendering strategy creates a cascade of secondary issues (focus loss, unnecessary network requests, broken keyboard interaction) that would each be small fixes individually but stem from a single architectural choice. The lack of version control and tests means there is no safety net for addressing any of these findings.

## Coverage Gaps

I did not audit browser compatibility beyond what is observable from CSS syntax (e.g., `color-mix(in oklch, ...)` requires modern browsers). I did not test actual runtime behavior, popup blocker interaction, or mobile rendering. I did not audit the inline SVG icon for correctness (the BV emblem at line 1587 is ~4KB of SVG paths). The "dependencies" category is minimal by design — the project intentionally has zero npm dependencies, so supply-chain analysis does not apply beyond the two external Google services already noted.

## Recommended First Steps

Start with findings 1 (javascript: URL injection) and 8 (session tokens in default URLs) — both are security issues with effort rating 1, meaning they can be fixed in minutes with high confidence. Then tackle finding 4 (git init) to establish version control before making further changes, since every subsequent improvement benefits from having a revertible history.
