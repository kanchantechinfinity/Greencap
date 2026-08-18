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

## 2026-08-17 — Reverted classic-hero-as-background change; added third hero section (side video)

User had asked earlier to make the classic hero use its product image as a section background (`fa218b7`). Shortly after asking for a third section copying "the first image hero section as it is whole" (with a new video, `/Users/apple/Desktop/side.mp4`, in place of the image), the user said to revert that background change. Used `git revert --no-edit ec415c6 fa218b7` (`a3a947b`, `fe514e8`) rather than hand-editing, so the classic hero's original two-column layout (text left, image right, floating badge) came back exactly as it was, including its build-log entry being cleanly un-added.

**Mistake made and caught:** the third-section markup had already been drafted (uncommitted) against the *background-style* classic hero before the revert request landed. Stashed that draft (`git stash push -u`, `-u` because it included the untracked `assets/hero-video-3.mp4`) to get a clean tree for the revert, then re-drafted the third section from scratch against the *restored* two-column layout — correct, since "copy the first hero section as it is" now meant the two-column version. But the stash was then dropped (`git stash drop`) as just "obsolete," which silently deleted the untracked video file inside it too — `hero-video-3.mp4` went missing from disk even though the HTML still referenced it. Caught it during verification (video stuck at `readyState:0`/`networkState:3` — `NETWORK_NO_SOURCE` — in the browser preview) rather than assuming the file was fine; re-copied `side.mp4` from the Desktop to `assets/hero-video-3.mp4` (verified with `md5` that the copy matches the source) before committing (`6381aa6`).

**Takeaway:** `git stash -u` sweeps untracked files into the stash; dropping that stash later deletes them for good, not just the tracked changes. When a stash held any untracked file (a build artifact, a new image/video) that's still wanted, re-copy or re-create it after dropping — don't assume "drop" only discards tracked edits.

**New section:** third hero (`<!-- Hero Section (Side Video) -->`) is a straight duplicate of the classic hero's two-column markup, video (`assets/hero-video-3.mp4`, 1920x1080, ~5.06s) swapping in for the `<img>` in the right column, `autoplay muted loop playsinline` like the other hero video. IDs renamed to `-side` suffix (`typewriter-headline-side`, `typewriter-subtext-side`) to avoid collisions. The classic hero's headline+subtext loop was factored into a shared `startHeadlineSubtextLoop(headlineId, subtextId)` function so both the classic and side sections reuse the same loop code instead of duplicating it a third time.

Verified `assets/hero-video-3.mp4` plays correctly in isolation (own test page, `readyState:4`) — it only fails to reach `readyState:4` when loaded as the *second* autoplay video on the full page in this sandboxed preview browser, which is a concurrency quirk of the preview tool itself (real browsers handle multiple autoplay-muted videos fine); not expected to be an issue on the actual deployed GitHub Pages site.

## 2026-08-17 — Side-video hero: match column height, slow playback (`62f3910`)

User flagged (via screenshot) that the side-video hero's video was shorter than the text column beside it and asked for the video to fill the same height, plus a slight slowdown.

- Section row's `align-items` changed from `center` (the `items-center` utility) to `stretch` via inline `style="align-items:stretch;"` — **not** a Tailwind `items-stretch` class, since that utility doesn't exist in the compiled stylesheet (only `.items-start`/`.items-center` are compiled; see [greencap_static_tailwind_css](../../../.claude/projects/-Users-apple-Desktop-go-high-level/memory/greencap_static_tailwind_css.md) — checked with `grep -o '\.items-[a-z]*{[^}]*}'` before using it, would have silently no-opped otherwise).
- Video column div gets `min-height:320px` (mobile fallback, since it's `flex-col` below `md` and the video is now absolutely positioned so it no longer contributes intrinsic height to the column).
- Video itself switched from `w-full h-auto` to `position:absolute;inset:0` + `w-full h-full object-cover` — fills whatever height the flex-stretch (desktop) or `min-height` (mobile) gives its parent, cropping via `object-cover` instead of preserving native aspect ratio.
- Added `id="hero-video-side"`; script sets `playbackRate = 0.75` on load for a slight slow-motion feel, without changing pitch/audio (video is muted anyway).

Verified with `getBoundingClientRect()` on both columns at 1280px width: text column and video column both measure exactly 536px tall — confirms the stretch is real, not just visually close.

## 2026-08-17 — Classic hero: swapped in supplied lifestyle photo (`96642a5`)

User attached an image (two women laughing at a phone, cream background) and asked to use it in "the first section very first hero section." The attachment wasn't a file path — found it by checking `ls -lat ~/Desktop/` for the most recently modified file, which turned out to be `GREEN CAP CREATIVES_21.png` (modified seconds before the message arrived); read it back to visually confirm it matched what was attached before using it.

Copied to `assets/hero-classic.png`, swapped into the classic hero's `<img src>` at index.html:146 only — left the *second* occurrence of the old Google-hosted vial-image URL (a different, unrelated product card further down the page, ~line 497) untouched, since the request was scoped to "the first hero section."

**Aside — deploy delay looked like a bug:** right after this push the user reported "it did not happen." The site really hadn't updated: `gh api repos/kanchantechinfinity/Greencap/pages/builds` showed the legacy GitHub Pages builder erroring intermittently — including on a *pure text* `BUILD_LOG.md`-only commit, which rules out anything content-specific. Most likely cause: GitHub Pages' legacy (Jekyll) builder has an undocumented soft rate limit (roughly 10 builds/hour) and this session pushed far more than that in under two hours. Waited ~20s and rechecked `pages/builds/latest` — the next build succeeded and picked up all pending commits (Pages deploys the latest successfully-built tree, not necessarily the exact commit that triggered it). **Takeaway:** after a push, if the live site doesn't reflect it, check `gh api repos/<owner>/<repo>/pages/builds/latest` (and `/builds` for history) before assuming the HTML/asset change itself is wrong — a red herring when this session is pushing at high frequency.

## 2026-08-17 — Fixed classic hero image height; deleted the video hero section (`5b47921`)

Two issues from a screenshot: (1) the new lifestyle photo in the classic hero is portrait-oriented and was rendered at `w-full h-auto`, so it scaled far taller than the text column — not actually cropped by CSS, but requiring a lot of scrolling and looking like it "cuts" at the viewport edge. (2) explicit instruction to delete the second hero section (the video-in-a-gap one with the growing/typed headline).

- Classic hero row: `items-center` → inline `style="align-items:stretch"` (same reasoning as the side-video hero fix earlier — `.items-stretch` isn't in the compiled Tailwind stylesheet, so it has to be inline, not a class). Image column gets `min-height:320px` mobile fallback; `<img>` switched to `position:absolute;inset:0;w-full h-full object-cover` so it crops to fill exactly the height the stretched row gives it, matching the text column (verified 536px = 536px at 1280px width) instead of running long.
- Text column added `justify-center` so its content stays vertically centered now that the column is stretched to the taller row height instead of being sized by its own content.
- Deleted the entire `<!-- Hero Section (Video) -->` block, its dedicated `timeupdate`-driven scale/type script block, the now-orphaned `.hero-gap-btn` CSS rule, and the unused `assets/hero-video-2.mp4` (`git rm`). Confirmed via `getElementById` that `typewriter-headline-video` / `hero-video` / `hero-headline-gate` no longer exist anywhere in the DOM. Two hero sections remain: classic (photo) and side-video (video in the same two-column layout).

## 2026-08-17 — Fixed image/video height fluctuating during typing (`8772bca`)

User caught (via screenshot) that the stretched image/video kept resizing while the typewriter typed/erased the subtext — because `align-items:stretch` recomputes the flex line's cross-size on every DOM mutation, and the text column's natural height changes as the subtext gains/loses characters (and therefore wrapped lines).

Fix: measure the text column once at its **fully-typed** size and lock that in as `min-height` before the typing loop starts, so the column (and the stretched sibling) never resizes mid-animation — `lockTextColumnHeight()` temporarily sets both the headline span and subtext to their full `data-typewriter` text, reads `getBoundingClientRect().height`, restores the previous (in-progress) text, then applies the measured value as `col.style.minHeight`. Re-runs on window `resize` so a breakpoint change gets a fresh measurement.

Two bugs caught and fixed while verifying with a `setInterval` sampler rather than trusting the code by eye:
1. **Wrong element locked.** `headline` is the `<span id="typewriter-headline-...">` inside the `<h1>`, not the column — `headline.parentElement` pointed at the `<h1>` itself, so the fix was locking a single element's height (which barely mattered) instead of the actual flex column. Fixed to `headline.closest('h1').parentElement`.
2. **Web-font race.** Measuring immediately on `DOMContentLoaded` sometimes ran before the custom fonts (Libre Caslon Text / Hanken Grotesk) finished loading, so the measurement used fallback-font metrics and wrapped to fewer lines than the real font would — locking in a height that was too short, which then visibly grew once the real font swapped in. Fixed by measuring inside `document.fonts.ready.then(...)` instead of synchronously.

Verified with a 300ms-interval sampler logging `{textColH, imgColH, subtextLength}` across a full type→erase cycle at both ~674px and 1280px widths: `textColH` and `imgColH` both stay pinned at a single constant value (e.g. 536px/536px at 1280px) for every sample, confirming the fix rather than assuming it from the code.

## 2026-08-17 — Removed the side-video hero section (`5f77c11`)

User asked to remove the remaining extra hero section (the two-column layout with the vials video, added earlier this session as "copy the first hero as-is with video instead of the image"). Only the classic (lifestyle-photo) hero remains now.

Deleted the `<!-- Hero Section (Side Video) -->` block (`sed -i '' '154,196d'`), its `startHeadlineSubtextLoop('typewriter-headline-side', ...)` call, and the `sideVideo`/`playbackRate` block in the script; `git rm`'d the now-unused `assets/hero-video-3.mp4`. Confirmed with `grep` that no `-side` id or `hero-video-side` reference remains anywhere in the file. Site now has a single hero section.

## 2026-08-17 — "How It Works" step cards cleanup (`8988273`)

Three small fixes from a screenshot of the 4-card step section:
- Removed the large decorative background numerals (`<div class="absolute -top-6 -left-6 text-8xl ... opacity-50">01</div>` etc.) from all four cards, not just the two the user explicitly named — same decorative element repeated, no reason to leave 03/04 behind.
- Card 4 ("Track Your Progress"): the closing sentence "Our support team is always available by text or phone." was its own `<p>` in `text-deep-forest`, inconsistent with the rest of the card's body copy (`text-secondary`). Merged it into the main paragraph so it inherits the same `font-body-md text-secondary` styling as every other card's description text.
- The phone number was plain text with a 📞 emoji under that sentence. Replaced with a pill matching the `bg-surface-container ... rounded-full` tag style used for "Delivery or Pickup" etc. elsewhere in the same section, swapping the emoji for the `call` Material Symbol icon to match the other pills' icon+label pattern exactly.

## 2026-08-17 — Features banner one-line fix + site-wide scroll-reveal animation (`0917804`)

**Features banner:** the 5-item trust-badge row (HIPAA Compliant / No Insurance Required / Discreet Next-Day Ship / Licensed TX Providers / FDA-Approved Ingredients) used `flex flex-wrap justify-between`, which wrapped to 4+1 across two rows at normal desktop widths. Fixed with a dedicated `.features-banner` CSS class (grid, not flex) in the custom `<style>` block — 2 columns on mobile, 3 at `md`, 5 at `lg`+ — rather than reaching for a Tailwind `grid-cols-5`/`lg:grid-cols-5` utility, since neither exists in the compiled stylesheet (checked first; see [greencap_static_tailwind_css](../../../.claude/projects/-Users-apple-Desktop-go-high-level/memory/greencap_static_tailwind_css.md)).

**Scroll-reveal animation:** user asked (after a short exploratory back-and-forth on scope/tradeoffs) for animation across the whole site. Implemented as a `.scroll-reveal`/`.is-visible` CSS pair (opacity + `translateY(24px)→0`, 0.6s, respects `prefers-reduced-motion`) plus a single `IntersectionObserver` in a new `<script>` block that runs on `DOMContentLoaded`, adds `.scroll-reveal` to every `<section>` **except the first** (the hero — already animated via its own typewriter effect and visible immediately on load, so gating it behind scroll-reveal risked a flash-of-invisible-content), and un-observes each section once it's revealed (one-time reveal, not re-hidden on scroll-up). Deliberately JS-driven rather than hand-adding a class to every section in the markup — zero risk of missing a section or fighting existing classes, and trivially reversible (delete the one script block).

Verified live: `document.querySelectorAll('.scroll-reveal.is-visible').length` matched the sections already in/above the viewport at test scroll position, and the hero section correctly has neither class.

## 2026-08-17 — Features banner redone as flip cards (`a374497`)

User pushed back hard ("NO ITS BECOMING TOO MUCH CHAOS") on the one-line grid fix from the previous entry — too cluttered with icon+label+sub-label crammed into 5 narrow columns. Asked instead for flip cards: icon-only front, full content on the back, revealed on hover, front face in the site's green.

- Each of the 5 items is now a `.flip-card` > `.flip-card-inner` > `.flip-card-front` / `.flip-card-back` pair using a standard CSS 3D-flip (`perspective` + `transform-style: preserve-3d` + `backface-visibility: hidden` + `rotateY(180deg)` on hover) — hand-written in the custom `<style>` block since there's no Tailwind utility for 3D transforms in this stack at all (not a "missing from the compiled sheet" case this time, just not something Tailwind ships).
- Front face uses the existing `bg-deep-forest` / `text-white` utility classes (the site's brand green) rather than a new hardcoded color, so it stays in sync with every other green element on the page automatically.
- Added a `click` handler (new tiny script block) that toggles an `.is-flipped` class, so the flip also works via tap on touch devices that don't have a real `:hover` state — CSS rule is `.flip-card:hover .flip-card-inner, .flip-card.is-flipped .flip-card-inner { transform: rotateY(180deg); }`.
- `prefers-reduced-motion: reduce` disables the flip transition (snaps instantly) same as the scroll-reveal animation.
- Kept the `.features-banner` grid wrapper (2/3/5 responsive columns) from the previous fix — still the right layout, just each grid cell is now a flip card instead of an inline icon+text row.

Verified via hover (screenshot: front cards flip to reveal white back face with label + description) and via a scripted `.click()` + `classList.contains('is-flipped')` check for the tap path.
