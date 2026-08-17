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

## 2026-08-17 — Hero overlay slimmed to headline-only, gated to video timing

User asked to: shrink the overlay text further so it visibly sits between the vials, remove the subtext paragraph / CTA buttons / "Licensed Texas Providers" badge entirely, and only show the headline once the vials are side-by-side (not while still touching).

**Changes (`0db6ef7`):**
- Removed the badge, `#typewriter-subtext` paragraph, both CTA links, and the trust-icons row from the hero overlay. Only the `<h1>` (with the `#typewriter-headline` typed span) remains.
- Headline sized with `style="font-size:clamp(0.55rem, 2.6vw, 1.4rem)"` and its wrapper `div#hero-headline-gate` set to `width:26%` — both inline styles, deliberately **not** new Tailwind utility classes, since [greencap_static_tailwind_css](../../../.claude/projects/-Users-apple-Desktop-go-high-level/memory/greencap_static_tailwind_css.md) means any unrecognized class silently no-ops. `clamp()`/percentage width need no breakpoint variants and scale continuously with the card's actual rendered width.
- Video given `id="hero-video"`. JS listens to its `timeupdate` event and sets `#hero-headline-gate`'s inline `opacity` to `1` once `currentTime >= 1.8` (vials visibly separated) and `0` below that (still touching) — `SIDE_BY_SIDE_AT` constant in the script, easy to retune if the separation timing ever looks off.
- Typewriter loop simplified to headline-only (type → hold 1.8s → erase → 0.4s pause → repeat); the subtext type/erase steps were removed since the element no longer exists.

**Verified:** opacity toggles correctly on natural playback (checked via a 2s `timeupdate` listener sampling `currentTime`/`opacity` together — confirms real coupling, not just plausible-looking code). Manual `currentTime =` jumps in devtools don't reflect live in a screenshot taken moments later because the muted video keeps playing/looping in the background during the round-trip — expected, not a bug.

## 2026-08-17 — Restored classic hero alongside the video hero for comparison (`aea8dd5`)

User wants to see both hero designs live on the page before picking one to keep. Pulled the original two-column (badge/headline/subtext/CTAs/trust-icons + product image) hero markup back from commit `32960f4` (the last commit before it was replaced) and reinserted it as its own `<section>` directly above the video hero — video hero is now the second hero-like section, right before the Trust Bar.

**ID collisions handled:** the restored classic section reuses the same typewriter pattern, so its elements were renamed to avoid clashing with the video hero's:
- Classic: `#typewriter-headline-classic`, `#typewriter-subtext-classic`
- Video: `#typewriter-headline-video` (was `#typewriter-headline`)

Script's single `DOMContentLoaded` handler now runs two independent loops — classic (type headline → type subtext → hold → erase both → repeat) and video (type headline only → hold → erase → repeat, gated to `#hero-video`'s `timeupdate` as before). Each loop no-ops safely if its elements aren't found, so either section can be deleted later without touching the other's code.

**Next step:** once the user picks one, delete the other section's markup and its corresponding loop block in the script (and drop `assets/hero-video.mp4` + the `hero-text-shadow`/`typewriter-cursor` CSS if the video hero is the one cut).

## 2026-08-17 — Swapped video hero clip; text now grows in sync with vial separation (`56ae659`, `b09e315`)

User supplied a second video (`/Users/apple/Desktop/001.mp4`, 1920x1080, ~5.04s) where the vials start touching center and end up pushed to the extreme left/right corners of the frame — same idea as the first clip but a much bigger final gap. Copied to `assets/hero-video-2.mp4`, swapped into the video hero's `<source>` (first clip `assets/hero-video.mp4` left in place, unused, in case the user wants to revert).

Inspected frames by seeking the `<video>` element and screenshotting (`currentTime` set via JS, `seeked` event awaited) rather than guessing timings: vials are still together through ~t=1.5s, mid-separation ~t=2–4s, essentially at their final corner position by ~t=4.2–4.3s, loop duration ~5.04s.

Two iterations on the reveal logic, both driven by `#hero-video`'s `timeupdate` (no `setTimeout`-based loop — everything is a function of `video.currentTime`, so it self-corrects every frame instead of drifting out of sync with the video like an independent timer would):
1. First pass (`56ae659`): binary gate — hidden/empty until `currentTime >= 4.2` (`CORNERS_AT`), then type once; reset when time wraps back below that on the next loop.
2. User asked (in Hinglish: *"jaise bottle dhur hote hai extreme end pe, wise text bade hote jane chahiye, smooth finish ke liye"* — "as the bottles move to the extreme ends, the text should grow bigger too, for a smooth finish") for continuous growth, not a sudden appearance. Replaced the binary gate (`b09e315`) with a progress ratio: `progress = clamp((currentTime - 1.5) / (duration - 1.5), 0, 1)`, driving `transform: scale()` from `0.6` to `1.6` every `timeupdate` tick, plus a short `0.15s` CSS transition to smooth between ticks. Typing still triggers once per cycle (`typedThisCycle` flag) right as `progress` first exceeds `0.02`, so the reveal is: fade/scale in small partway through separation → keeps growing → hits max size (1.6x) exactly as vials reach the corners → resets to hidden/`scale(0.6)` when the loop wraps.

Verified by sampling `{currentTime, opacity, transform}` on a 400ms interval across a full loop — confirms monotonic scale growth (0.6 → ~1.6) synced to the video's own clock, not a fixed-duration animation that could drift.

## 2026-08-17 — Video hero: headline line break + CTA buttons added (`1e4d345`)

Ambiguous request ("bring 100% online in another line... also wheras that buttons") resolved via clarifying question: applies to the **video hero** (not classic), and "buttons" meant *add* CTA buttons to the video hero (it had none since the para/buttons/badge strip-down earlier this session), not modify the classic hero's existing ones.

- Added a `<br>` between "Texas Telehealth Prescriptions," and the typed "100% Online" span so they're always on separate lines, regardless of container width.
- Added "Start Free Assessment" and phone-number CTA links below the headline, inside `#hero-headline-gate` — so they inherit the same fade-in + `scale(0.6→1.6)` growth and only appear once the vials start separating, same as the headline.
- New `.hero-gap-btn` class (clamp-based font-size/padding) keeps the buttons legible at the small end of the scale range without overflowing the narrow gap early in the loop. Widened `#hero-headline-gate` from `width:26%` to `32%` to give the buttons (which are `white-space:nowrap`) room.
- Used `bg-white bg-opacity-90 backdrop-blur-md` for the secondary (phone) button — confirmed `.bg-opacity-90` exists in the compiled stylesheet before using it (per [greencap_static_tailwind_css](../../../.claude/projects/-Users-apple-Desktop-go-high-level/memory/greencap_static_tailwind_css.md)); did **not** reuse `bg-white/80` from the old (already-removed) video-hero button since that arbitrary-opacity class was never confirmed to exist in the compiled sheet.

## 2026-08-17 — Classic hero restructured to use the image as section background (`fa218b7`)

User pointed out the classic hero still used the original two-column layout (text in a `w-full md:w-1/2` div, image in a separate `w-full md:w-1/2` div next to it) while the video hero used its media as a true section background with content overlaid on top — asked for the classic hero to match that pattern.

Rebuilt classic hero markup 1:1 against the video hero's structure: `<img>` (was inside its own column div with a decorative rotated `bg-sage-tint` div behind it — both removed) is now `w-full h-auto block` directly inside the `relative rounded-[40px] overflow-hidden` card, exactly like `<video id="hero-video">`. All prior left-column content (badge, headline, subtext, CTAs, trust icons) moved into an `absolute inset-0 flex items-center justify-center` overlay, centered instead of left-aligned, with `hero-text-shadow` added to each text element for legibility against the photo (same technique already used on the video hero). Floating "Trusted by" badge kept, anchored to the card's bottom-left corner.

Both hero sections are now structurally identical (image vs. video as the only difference) — makes it trivial to delete either one later without leaving orphaned layout patterns behind.
