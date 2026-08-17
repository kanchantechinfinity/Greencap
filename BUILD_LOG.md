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

## 2026-08-17 — Hero typewriter animation + product video

**Typewriter effect:** Added a typing animation to the hero headline span ("100% Online") and subtext paragraph — pure JS (`typeText`/`eraseText` in a `<script>` before `</body>`), driven by `data-typewriter` attributes on `#typewriter-headline` / `#typewriter-subtext`. Iterated through 3 versions per user feedback:
1. Type once on load, cursor stops after (`is-done` class → `animation:none; opacity:0`).
2. (No change needed — confirmed already met "type once, stay still".)
3. Changed to infinite loop: type both lines → hold 1.8s → erase both (reverse order) → 0.4s pause → repeat. Cursor blinks continuously now (`is-done` logic removed).

**Hero video:** User supplied `hf_20260817_..._c4a8f50d....mp4` (1920x1080, ~5s, two vials that start touching and separate over the clip, silent). Copied into repo at `assets/hero-video.mp4` (~5.4MB) since `<video>` sources must be same-origin-reachable relative paths for GitHub Pages (and for local preview — the browser preview tool blocks `file://` video sources outside the project directory, confirmed via `networkState`/`error.code 4`).

Per user's explicit choice (asked via clarifying question): rebuilt the hero as a **full-width video** (not a two-column text+image split) with the full headline+subtext typed out centered over the gap between the vials. Implementation:
- `<video autoplay muted loop playsinline>` inside a `relative rounded-[40px] overflow-hidden` card, `w-full h-auto` (no `object-cover`) so it always keeps its native 16:9 aspect and is never cropped.
- Overlay content in an `absolute inset-0 flex items-center justify-center` wrapper, centered on the video's exact box.
- First pass wrapped the text in an opaque `bg-white/90 backdrop-blur-md` card for legibility — user asked to remove it so the bottles stay visible; replaced with a `hero-text-shadow` utility (soft white text-shadow glow) on the heading/subtext/badge/trust-icons instead, and shrank the whole block (`max-w-2xl`→`max-w-md`, larger paddings/margins/font sizes trimmed down, `text-display-lg`→`text-headline-lg`).
- Floating "Trusted by 2,400+ Texans" badge kept, anchored to the video card's bottom-left corner (`hidden md:flex` — desktop only, to avoid crowding on mobile).

**Commits:**
- `d410d6d` — one-shot typewriter (headline then subtext, cursor stops)
- `32960f4` — looping typewriter (type/erase repeat)
- `dd701d3` — hero video replacing static image, text overlay in vial gap, later slimmed down per feedback (white panel removed, sizes reduced) — all folded into this single commit

**Gotcha for next time:** the vials start touching (no visible gap) for the first ~1.5s of each loop and only fully separate by ~2s in; the overlay text will briefly sit across the vials early in each loop cycle. User was told this tradeoff and accepted it when choosing "full headline + subtext" over a shorter phrase.
