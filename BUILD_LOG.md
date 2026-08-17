# Build Log

## 2026-08-17 — Repo connected + FAQ heading overlap fix

**Connected:** Cloned `kanchantechinfinity/Greencap` (GitHub Pages source for kanchantechinfinity.github.io/Greencap) locally for direct edit/commit/push access.

**Bug:** FAQ section heading ("Every reason not to start — answered.") overlapped the FAQ answer cards on mobile/tablet viewports.

**Root cause:** The heading wrapper had unprefixed `sticky top-32` classes, so `position: sticky` applied at all viewport widths. The section only switches to a two-column layout at `lg:flex-row` — below that it's a single stacked column (`flex-col`), so the sticky heading stayed pinned while the cards column scrolled underneath it, causing visual overlap.

**Fix attempt 1 (incomplete):** Changed classes to `lg:sticky lg:top-32` in the HTML. This had no effect — `index.html` uses a static pre-compiled Tailwind stylesheet (`<style id="tailwind-compiled">`, no CDN/build step), and `.lg\:sticky`/`.lg\:top-32` rules didn't exist in the compiled CSS. Unknown classes are silently ignored by the browser.

**Fix attempt 2 (actual fix):** Added `.lg\:sticky{position:sticky}.lg\:top-32{top:8rem}` inside the existing `@media (min-width:1024px)` block in the compiled `<style>` tag. Verified via computed styles: `position: static` at 375px and 674px widths, `position: sticky; top: 128px` at 1280px.

**Commits:**
- `a48e05b` — HTML class change (attempt 1, insufficient alone)
- `62a3c10` — compiled CSS rule addition (completes the fix)

**Takeaway:** Any future `lg:`/`md:`/etc. class introduced in this file must be checked against the compiled `<style>` block — see memory `greencap_static_tailwind_css`.
