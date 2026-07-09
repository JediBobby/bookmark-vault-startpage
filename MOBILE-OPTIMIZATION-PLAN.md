# Mobile + iPad (M4 Air) UI Optimization Plan — Bookmark Vault 2.0

All changes are in `index.html` (single-file rule is a hard constraint — no new assets, no build step).
Current responsive breakpoints: `@media (max-width: 860px)` (~line 684), `@media (max-width: 560px)` (~line 693), `@media (hover: none)` (~line 702).

## Target viewports

| Device | Portrait | Landscape |
|---|---|---|
| Phone | ~360–430px | ~740–930px |
| iPad Air M4 11" | 820px | 1180px |
| iPad Air M4 13" | 1024px | 1366px |

Note the overlaps: phone-landscape and iPad-11-portrait both land in the 740–930px band, so use `(hover: none)`/`(pointer: coarse)` to distinguish touch behavior from layout width, and don't treat 860px as "phone only".

## 1. Favorites rail → icon-only tiles on phones (user's explicit request)

- In the `@media (max-width: 560px)` block:
  - Hide `.fav-name` (`display: none`).
  - Change `.fav` from a 160px-wide pill to a square icon tile: `width: var(--tile-h)` (or ~52px fixed), `justify-content: center`, `padding: 0`.
  - Enlarge `.fav-ico` to ~26–28px so the icon carries the tile.
  - Keep the `title` attribute (already set on the `<a class="fav">` in the render at ~line 1173) so long-press/tooltip still reveals the name — no JS change needed.
  - Keep `.fav-key` number badges but shrink/reposition (they may be clutter on touch since 1–9 shortcuts are keyboard-only; consider hiding them under `(hover: none) and (max-width: 560px)`).
- With icon-only tiles, most favorite sets will fit without scrolling; keep the rail's `overflow-x: auto` as fallback. Optionally switch the rail to `flex-wrap: wrap` on phones so overflow wraps to a second row instead of hiding off-screen — decide by feel, wrapping is likely nicer on a start page.
- Do NOT apply icon-only at 860px — iPad portrait (820px) has room for named tiles; scope it to ≤560px (or ≤600px).

## 2. Top bar on phones

- At ≤560px the topbar currently keeps brand emblem + search + all action buttons in one row; verify it fits at 360px. If tight:
  - Reduce `.topbar-inner` gap, shrink `.brand-emblem` to ~36px.
  - Consider collapsing secondary top actions (theme, settings, etc.) behind the existing settings entry point or reduce `iconbtn` horizontal padding.
- Search input: font-size must be ≥16px on iOS (`0.95rem` ≈ 15.2px — bump to `1rem` at coarse-pointer/phone widths) to prevent iOS Safari auto-zoom on focus.

## 3. Touch ergonomics (phones + iPad)

- Under `@media (pointer: coarse)`:
  - Minimum 44×44px effective hit areas: `.iconbtn` (2.4rem ≈ 38px), `.act` row buttons in bookmark cards, `.search-clear` (1.7rem ≈ 27px — too small), `.col-tools` buttons. Bump heights/min-widths to 2.75rem or add padding.
  - `.p-item` in `v-list` view is 30px tall and `v-chip` 28px — increase to ≥40px on coarse pointers.
- The existing `@media (hover: none)` block already neutralizes hover transforms — extend it to add `:active` feedback (e.g. slight scale/background) so taps feel responsive.
- Hover-revealed controls: audit for any UI that only appears on `:hover` (e.g. card tools/actions). On touch they must be always visible or reachable via the detail view — make them always visible at reduced opacity under `(hover: none)`.

## 4. iPad-specific layout (both orientations)

- **Portrait 820/1024px:** falls into the 860px breakpoint for 11" but not 13". Verify `.collections` `minmax(300px,1fr)` gives 2 columns at 820px (it does: 2×300+gap ≈ 800 within ~756px content width → actually yields 2 cols only above ~630px content; confirm visually). Ensure the 860px block doesn't hide things iPads have room for — e.g. keep `.brand-name` and `.iconbtn .lbl` visible between 640–860px if space allows, or accept the current behavior.
- **Landscape 1180/1366px:** full desktop layout; verify `max-width: 1500px` wrap and `data-cols="wide"` grid look right — likely no changes needed beyond touch-target rules (iPad is `pointer: coarse` at desktop widths, which is exactly why touch fixes must NOT be width-gated).
- Safe areas: add `viewport-fit=cover` to the meta viewport and pad the sticky `.topbar` / bottom of `main` with `env(safe-area-inset-*)` so the PWA (it has a manifest + apple-touch-icon, ~line 2274+) respects the iPhone notch/home bar and iPad rounded corners when installed.

## 5. Modals, drawers, palette on small screens

- `.modal` (max-width 460px): on phones ensure it isn't cramped — full-width with margin, `max-height` using `dvh` not `vh` (iOS URL-bar issue), internal scroll. The 560px block already stacks `.modal-actions`.
- `.drawer` is `min(400px, 92vw)` — fine, but ensure its body scrolls with `-webkit-overflow-scrolling` behavior and add safe-area bottom padding.
- Command palette: check its positioning/height on phones with the software keyboard open; prefer `dvh` units and top-anchored placement.
- Detail view: `.detail-title` 1.6rem and `.detail-badge` 54px could shrink slightly at ≤560px.

## 6. Verification checklist (use preview server + preview_resize)

1. 390×844 (phone portrait): favorites are icon-only squares, no horizontal page scroll, topbar fits, search focus doesn't zoom (16px font).
2. 844×390 (phone landscape): rail + collections usable.
3. 820×1180 and 1180×820 (iPad 11" both orientations): collections 2–3 cols, named favorites retained, touch targets ≥44px.
4. 1024×1366 / 1366×1024 (iPad 13"): desktop-like layout intact.
5. Dark + light themes at each size (preview_resize colorScheme).
6. Open add-bookmark modal + notes drawer + command palette at 390px width.

## Constraints / reminders

- Single file, no build step, no external assets.
- Branch off `bookmark-vault-2.0` or continue on it per user direction; commit/push only when asked.
- No tests needed (project convention).
