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

**Intentional layer** — for library-driven motion, prefer degrading over deleting. Framer Motion's
`<MotionConfig reducedMotion="user">` drops transforms while keeping opacity, so reveals still
communicate arrival without vestibular movement. Users who set the flag want *less* motion, not a
static page with no feedback.

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

## Scroll-driven motion

- **Smooth scroll (Lenis et al.)** — set `html { scroll-behavior: auto }` and `.lenis-smooth
  { scroll-behavior: auto !important }` or native smooth-scroll fights the library.
- **Scrub, do not pin.** Driving video `currentTime` or a progress value from scroll keeps the user in
  control. Multiple pinned sections fight each other and break scroll position restore.
- Gate scrub on `prefers-reduced-motion` — fall back to a poster frame.

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
