# Motion

Motion separates designed work from generated work faster than any other dimension, because the
default is so weak: one flat opacity fade, applied to everything, with no off switch.

---

## Tokens: one curve, three durations

```css
--ease:      cubic-bezier(0.22, 1, 0.36, 1);   /* ease-out-expo: fast start, long settle */
--dur-micro: 150ms;   /* press, hover, tiny state flips */
--dur-ui:    220ms;   /* color, shadow, background changes */
--dur-enter: 300ms;   /* elements arriving on screen */
```

Mirror them in JS so CSS and your animation library never drift:

```ts
export const EASE_OUT_EXPO = [0.22, 1, 0.36, 1] as const;
```

**One curve for everything.** Multiple easings across a site is the motion equivalent of multiple
accent colors. Ease-out is right for almost everything — things should decelerate into place. Reserve
ease-in for exits only.

**Durations.** Under 100ms reads as an instant snap; over ~400ms feels laggy on repeat interactions.
Hover and press live at 150ms because they repeat constantly and any lag is felt as sluggishness.

---

## Animate transform and opacity. Only.

These two are the only properties the compositor can animate without layout or paint work.

| Never | Instead |
| --- | --- |
| `top` / `left` | `transform: translate3d()` |
| `width` / `height` | `transform: scale()` |
| `margin` | `transform: translate3d()` |
| `box-shadow` in a loop | cross-fade two stacked pseudo-elements |

Use `translate3d(x, y, 0)` rather than `translate(x, y)` for a continuous animation — the z component
promotes to its own layer. Pair with `will-change: transform` on genuinely continuous animations only;
on everything else it wastes memory and can *hurt*.

---

## The site reveal: one, shared

Define it once. Every section header and staggered child uses it. A **rise plus a clearing blur** costs
the same as a fade and reads as deliberate:

```ts
export const REVEAL_INITIAL   = { opacity: 0, y: 16, filter: "blur(6px)" } as const;
export const REVEAL_ANIMATE   = { opacity: 1, y: 0,  filter: "blur(0px)" } as const;
export const REVEAL_TRANSITION = { duration: 0.55, ease: EASE_OUT_EXPO } as const;

/** Fire slightly before fully in frame, once. */
export const REVEAL_VIEWPORT = { once: true, margin: "-80px" } as const;
```

- `y: 16` — enough to perceive, not enough to feel like the page is assembling itself.
- `blur(6px)` clearing to `0` — the "focus pulling in" quality. This is the part that reads as craft.
- `once: true` — replaying reveals on every scroll-by is actively irritating.
- `margin: "-80px"` — content is already settled by the time it is properly in view.

Stagger children by 60–80ms. More than ~5 staggered items and the last one arrives late enough to feel broken.

---

## Continuous animation: the pause contract

Any looping motion — marquee, ticker, ambient gradient — must pause on **hover, keyboard focus,
touch-hold, and when off-screen**, and stop dead under reduced motion. WCAG 2.2.2 requires a
mechanism to pause anything auto-moving for more than 5 seconds.

### The marquee, done properly

Duplicate the content into two identical sets and translate `-50%` — the loop is then seamless with
no JS measuring widths.

```css
.marquee {
  position: relative;
  overflow: hidden;
  width: 100%;
  /* Edges fade instead of clipping mid-card */
  -webkit-mask-image: linear-gradient(90deg, transparent 0%, #000 3%, #000 97%, transparent 100%);
          mask-image: linear-gradient(90deg, transparent 0%, #000 3%, #000 97%, transparent 100%);
}

.marquee__track {
  display: flex;
  width: max-content;
  will-change: transform;
  animation: marquee-x 42s linear infinite;
}

.marquee__set { display: flex; gap: 1.5rem; padding-right: 1.5rem; }

/* The pause contract */
.marquee:hover              .marquee__track,
.marquee:focus-within       .marquee__track,
.marquee[data-touch-paused="true"] .marquee__track,
.marquee[data-offscreen="true"]    .marquee__track {
  animation-play-state: paused;
}

/* Slower on mobile — the same speed feels frantic in a narrow column */
@media (max-width: 767.98px) {
  .marquee__track { animation-duration: 56s; }
}

@media (prefers-reduced-motion: reduce) {
  .marquee__track { animation: none; }
}

@keyframes marquee-x {
  from { transform: translate3d(0, 0, 0); }
  to   { transform: translate3d(-50%, 0, 0); }
}
```

Four details worth keeping:

1. **`linear`, not eased.** A continuous loop with an ease curve visibly stutters at the seam.
2. **The mask gradient.** Hard-clipped edges look like an overflow bug; a 3% fade looks intentional.
3. **`data-offscreen`** — driven by `IntersectionObserver`. Do not burn compositor cycles on an
   animation nobody can see.
4. **Mobile duration is longer.** Same px/sec in a narrower viewport reads as much faster.

Make the region keyboard-focusable so `focus-within` can actually fire, and give it a visible ring:

```tsx
<div data-reviews-marquee tabIndex={0} aria-label="Customer reviews, auto-scrolling. Focus to pause.">
```

```css
[data-reviews-marquee]:focus-visible {
  outline: 2px solid var(--accent-500);
  outline-offset: 4px;
  border-radius: 4px;
}
```

Touch has no hover, so hold-to-pause needs wiring explicitly:

```tsx
onTouchStart={() => el.setAttribute("data-touch-paused", "true")}
onTouchEnd={()   => el.removeAttribute("data-touch-paused")}
```

---

## Reduced motion: two layers

**Global safety net** — nothing can slip past:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

**Intentional layer** — for library-driven motion, prefer degrading over deleting. Users who set the
flag want *less* motion, not a static page with no feedback: drop the translation, keep the opacity,
and the reveal still communicates arrival without vestibular movement.

Whether your engine does this for you is a selection criterion, not an afterthought. Framer Motion
ships it (`<MotionConfig reducedMotion="user">` drops transforms and keeps opacity globally). Most
engines do not, and you wire the degradation yourself — see the mapping below.

Expose it as a hook for JS-driven decisions (skipping scroll-scrub, swapping video for poster):

```ts
export function usePrefersReducedMotion() {
  const [reduced, setReduced] = useState(false);
  useEffect(() => {
    const mq = window.matchMedia("(prefers-reduced-motion: reduce)");
    setReduced(mq.matches);
    const on = (e: MediaQueryListEvent) => setReduced(e.matches);
    mq.addEventListener("change", on);
    return () => mq.removeEventListener("change", on);
  }, []);
  return reduced;
}
```

---

## Driving this from a library

Everything above is a **contract**: one curve, three durations, transform and opacity only, a pause
affordance on anything that loops, and a reduced-motion path. The contract does not care which engine
satisfies it. What changes per engine is how much you get for free and how much you wire yourself.

**Reach for CSS first.** Hover, press, and state transitions want `transition`, not a library. A
keyframe loop with `animation-play-state` beats a JS ticker for a marquee — the compositor runs it off
the main thread and `prefers-reduced-motion` reaches it without any code. Add an engine when you need
sequencing, scroll-linking, physics, or interruptible tweens. Not before.

### Contract → mechanism

| Requirement | CSS | Framer Motion | GSAP | anime.js v4 |
| --- | --- | --- | --- | --- |
| One curve | `var(--ease)` | `ease` in a shared transition const | `defaults({ ease })` on a timeline | `ease` in shared params; **avoid mixing** the six ease families |
| Three durations | `var(--dur-*)` | shared transition const | `defaults({ duration })` | `duration` in shared params |
| Transform/opacity only | you must self-police | ditto | ditto | ditto — `animate()` takes *any* property |
| Reveal on enter | — | `whileInView` + `viewport.once` | `ScrollTrigger` `once: true` | `IntersectionObserver` → `animation.play()` |
| Pause when off-screen | `animation-play-state` + IO | `useInView` | `ScrollTrigger` toggles | IO → `.pause()` / `.play()` |
| Reduced motion | media query reaches it free | `<MotionConfig reducedMotion="user">` | `gsap.matchMedia()` | **nothing built in — wire it** |

The three right-hand columns share a property worth noticing: they will all happily animate `width`
and `left`. Only CSS makes the cheap path also the default path.

### anime.js v4 specifically

Capable and small (10KB, or 3KB for the WAAPI build). It also removes every guardrail this file
depends on, so if you pick it, adopt these four rules explicitly:

| Rule | Why anime.js needs it stated |
| --- | --- |
| Transform and opacity only | `animate(el, { width: 400 })` is as ergonomic as the transform version, and nothing warns you |
| One curve | It ships built-in eases, cubic Bézier, linear, steps, irregular, and spring — six families inviting per-animation improvisation. Pick one, put it in a shared params object |
| Pause off-screen | `autoplay` defaults to `true`, so animations burn frames before anyone scrolls to them. Gate with `autoplay: false` + `IntersectionObserver` |
| Reduced motion | The documentation contains no `prefers-reduced-motion` guidance. It is entirely on you |

```js
// Shared params — the single place the contract lives.
const CONTRACT = { duration: 300, ease: "cubicBezier(0.22, 1, 0.36, 1)" };
const reduced = matchMedia("(prefers-reduced-motion: reduce)").matches;

const reveal = animate(el, {
  ...CONTRACT,
  opacity: [0, 1],
  translateY: reduced ? 0 : [16, 0],   // degrade the movement, keep the fade
  autoplay: false,                      // ← never let it run unobserved
});

new IntersectionObserver(([e], io) => {
  if (!e.isIntersecting) return;
  reveal.play();
  io.disconnect();                      // once: true, by hand
}, { rootMargin: "-80px" }).observe(el);
```

Note how much of the contract is manual here versus the Framer snippet earlier in this file. That is
the actual trade: smaller bundle, more discipline required. Fine if you write the discipline down —
which is the whole thesis.

API reference: <https://animejs.com/documentation/animation/>. Treat it as mechanism lookup only; it
carries no design or accessibility guidance.

---

## Scroll-driven motion

Two decisions here, and both are commonly made by accident. Make them on purpose.

**Do you need a smooth-scroll library at all?** Lenis, Locomotive and friends replace native scrolling
with a rAF-interpolated transform. You get scroll-linked animation that never tears, and a page feel
you cannot get natively. You also inherit: broken `scroll-behavior`, anchor-link and scroll-restoration
quirks, accessibility risk if the interpolation lags input, and a hard dependency on the library
staying maintained. **Default to native.** Adopt one only when the design genuinely depends on
scroll-linked motion staying locked to the scrollbar.

If you do adopt one, stop native smooth-scroll from fighting it:

```css
html { scroll-behavior: auto; }
.lenis.lenis-smooth { scroll-behavior: auto !important; }
```

**Scrub, do not pin.** Driving a video `currentTime` or a progress value from scroll position keeps
the user in control of the page. Pinning takes that control away, and multiple pinned sections fight
each other and break scroll-position restore. GSAP's `ScrollTrigger` is the mature choice for scrub —
but note you are adding it *for scroll-linking*, not as a general animation engine. If it is only
doing reveals, an `IntersectionObserver` and CSS will do the same job for none of the bundle.

Gate any scrub on `prefers-reduced-motion` and fall back to a poster frame.

## Micro-interaction: the arrow link

Small, cheap, and unmistakably hand-made. The glyph travels independently of the label:

```css
.arrow { display: inline-block; transition: transform var(--dur-ui) var(--ease); will-change: transform; }
.arrow-link:hover .arrow,
.arrow-link:focus-visible .arrow,
.group:hover .arrow,
.group:focus-within .arrow { transform: translate3d(0.25rem, 0, 0); }
```

## Checks

- [ ] One easing curve, three duration tokens, no raw values in components
- [ ] Only `transform` / `opacity` animated
- [ ] One shared reveal — rise + blur, not a flat fade — with `once: true`
- [ ] Every loop pauses on hover / focus-within / touch / off-screen
- [ ] Looping regions are focusable, labelled, and have a focus ring
- [ ] `prefers-reduced-motion` honored globally *and* intentionally
- [ ] Continuous loops are `linear`; mobile durations lengthened
- [ ] `will-change` only on genuinely continuous animations
- [ ] Every animation library in the bundle earns its place — name what it does that CSS could not
- [ ] If the engine has no reduced-motion support, the degradation is written down, not assumed
- [ ] Nothing autoplays before it is on screen
