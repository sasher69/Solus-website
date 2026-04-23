# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Session Start — Required Greeting

At the start of every session, greet Seb with this exact message before doing anything:

> "Hi Seb — before we make any changes, please confirm:
>
> 1. WHAT ARE WE UPDATING TODAY?
>    [ ] Design only — layout, colours, sections, fonts, content (hero animation untouched)
>    [ ] Animation only — hero scroll, frame timing, lock behaviour (design untouched)
>    [ ] Both — confirm which order to do them in
>
> 2. HERO ANIMATION PROTECTION
>    hero-animation.js and both frame folders are locked unless you explicitly say:
>    'you have permission to edit the animation today'
>
> 3. CLAUDE.md MEMORY UPDATE
>    At the end of every session I will ask:
>    'Do you want me to update CLAUDE.md to reflect what we changed today?'
>    Always say yes so memory stays current.
>
> 4. BEFORE ANY MAJOR CHANGE
>    If I am about to edit index.html or any core file I will tell you exactly what I am changing and wait for confirmation before proceeding.
>
> 5. NEW CLAUDE DESIGN HANDOFF
>    If you bring in a new handoff bundle always remind me to say:
>    'Update design only — do not touch hero-animation.js or the frames folders'
>
> Please confirm which mode we are working in today before I touch anything."

---

## Session End — Required Prompt

At the end of every session ask:
> "Should I update CLAUDE.md with what we changed today?"

---

## Protected Files — Never Edit Without Explicit Permission

Seb must say **"you have permission to edit the animation today"** before touching any of these:

- `hero-animation.js`
- `assets/frames/` (all contents)
- `assets/frames-reverse/` (all contents)

---

## Project

Premium dark-theme sports nutrition website for **SOLUS** — a two-ingredient endurance carb mix brand based in Queensland, Australia. Plain HTML/CSS/JS, no frameworks. Deployed on Vercel via the `solus-website` GitHub repo (push to main triggers auto-deploy).

**Preview locally:**
```bash
npx serve .
# from C:\Users\vicas\OneDrive\Desktop\Solus-website
```

## Brand

| Token | Value |
|---|---|
| Background | `#0a0a0a` |
| Accent | Electric blue `#3B82F6` |
| Heading font | Monument Extended (local woff2/woff in `assets/`) |
| Body font | Inter |
| Typography | White throughout |
| Aesthetic | Minimal, athletic, premium, dark |

Product: **CARB MIX** · 1 kg · 20 serves · HASTA certified · Made in Australia

## Project Structure

```
index.html                  — entire site (HTML + CSS + JS in one file)
hero-animation.js           — scroll animation logic ← PROTECTED
assets/
  frames/                   — forward animation frames ← PROTECTED
  frames-reverse/           — reverse animation frames ← PROTECTED
  Solus-hero2.mp4           — hero video (forward)
  Solus-hero2-reverse.mp4   — hero video (reversed)
  MonumentExtended-Regular.woff2/.woff
```

## Smooth Scroll — Lenis

Lenis 1.3.23 is loaded via CDN in `<head>` (CSS + JS). Initialised at the top of the main `<script>` block, before any hero logic:

```js
const lenis = new Lenis({ lerp: 0.08, wheelMultiplier: 1.4, smoothWheel: true });
lenis.on('scroll', ScrollTrigger.update);
gsap.ticker.add((time) => { lenis.raf(time * 1000); });
gsap.ticker.lagSmoothing(0);
```

- `lenis.stop()` / `lenis.start()` replace manual `preventDefault` for the hero scroll lock — Lenis owns all scroll prevention
- `lenis.scrollTo(0, { immediate: true })` is used in `startReverse()` instead of `window.scrollTo`
- All wheel listeners during the locked phase are `passive: true` — no manual `e.preventDefault()` anywhere
- `onTouchMoveLocked` has been removed entirely (Lenis handles touch blocking when stopped)

> **Session 2026-04-22:** Added Lenis for smooth scroll. Rebuilt scroll lock to use lenis.stop()/lenis.start(). No state machine changes.

## Hero Animation Behaviour

- Page loads on frame 1, video still
- Scroll locked for 3 scroll inputs before the page releases
- On the 3rd scroll click the forward video plays natively at full speed; scroll unlocks **immediately** — user can browse while it plays
- Holds on the last frame when complete
- Reverse arms once the user has scrolled past the hero section + 200 px (`hasScrolledPast` flag); triggers when they return to `scrollY < 5`
- Reverse video plays, then resets to idle with the 3-click gate re-armed
- Logic is currently **inline in `index.html`** (bottom `<script>` block) using a 4-state machine: `idle → playing-forward → free → playing-reverse`
- Never modify this logic unless Seb explicitly grants permission

> **Session 2025-04-21:** Replaced choppy scroll-mapped playback with the state machine above. Key fix — video now plays via native `.play()` uninterrupted by scroll input, eliminating the choppiness.

## Navigation

Floating pill-style nav — always frosted glass, never full-width.

| Property | Value |
|---|---|
| Position | `fixed`, `top: 20px`, `left: 50% + translateX(-50%)` |
| Shape | `border-radius: 9999px`, `height: 48px`, `padding: 0 24px` |
| Background | `rgba(255,255,255,0.05)` + `backdrop-filter: blur(20px)` |
| Border | `1px solid rgba(255,255,255,0.1)` |
| z-index | `50` (above hero video, below any modal) |

- `.scrolled` class is toggled by JS scroll handler but is a visual no-op — pill is always frosted
- Mobile `≤ 640px`: `.nav-r` hides, `.nav-burger` (2-line → X) appears, padded to 44×44px touch target
- Tablet `641px–1024px`: full nav links visible, pill slightly smaller (46px height)
- Large desktop `1441px+`: pill scales up (52px height, wider gap, larger logo/link text)
- `#nav-mobile` dropdown panel (`border-radius: 20px`) appears below the pill when burger is open; closes on any link click **or tap outside**
- Burger toggle JS is at the bottom of the main `<script>` block — **no scroll listeners added**
- Outside-tap close: `document.addEventListener('click', ...)` checks `burger.contains` / `mobileNav.contains` — added after burger toggle code

> **Session 2026-04-21:** Replaced full-width edge nav with floating pill nav. Added hamburger + `#nav-mobile` panel for mobile. No scroll logic touched.
> **Session 2026-04-23:** Added outside-tap hamburger close. Added 44×44px touch target on burger. Nav scales for tablet and large desktop.

## Page Sections (DOM order)

`#nav` + `#nav-mobile` → `.hero-scroll` → `#ticker` → `#chapters` (300 vh sticky) → `#ratio` → `#research` → `#philosophy` → `#shop` → `footer`

**Pinned chapters** (`#chapters`): sticky container, 3 panels (`#cp0–cp2`) + dot indicators (`#dot0–dot2`). Active panel derived from scroll progress each animation frame.

**Scroll reveal**: `.reveal` elements use `IntersectionObserver` to add `.in`. Delay variants: `.d1`, `.d2`, `.d3`.

## Responsive Breakpoints

All responsive styles are **additive-only** — no existing desktop styles were modified. New `@media` blocks appended at the end of the `<style>` tag.

| Breakpoint | Range | Key behaviour |
|---|---|---|
| Extra-small mobile | `≤ 375px` | Tighter padding, smaller fonts, 44px nav |
| Mobile | `≤ 640px` | Hamburger shown, 44×44px touch target |
| Tablet (2-col restore) | `641px–1024px` | Chapters 2-col restored, tub visible, shop 2-col |
| Tablet portrait | `641px–834px` | Tub at ~168×208px, medium padding |
| Tablet landscape | `835px–1024px` | Tub at ~200×248px, larger padding |
| Desktop | `1025px–1440px` | **Unchanged — original styles** |
| Large desktop | `1441px–1920px` | 52px nav, 120px horizontal section padding |
| Ultra-wide | `1921px+` | Hero capped at 1920px, sections padded to avoid full-bleed text |
| Touch devices | `hover: none` + `pointer: coarse` | All hover effects disabled; active-state feedback added |
| Reduced motion | `prefers-reduced-motion: reduce` | Reveals instant, ticker slower, pulse rings static |

> **Session 2026-04-23:** Full responsive pass added. All changes are new media query blocks only — desktop (1025px+) confirmed unchanged.

## Rules

- Always preserve the hero section when updating design
- No frameworks — plain HTML/CSS/JS only
- CSS custom properties for all colours/fonts (defined on `:root`)
- Before any major change to `index.html` or core files: state exactly what is changing and wait for confirmation
