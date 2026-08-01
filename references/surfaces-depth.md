# Surfaces, Depth & Context Inversion

---

## Part 1 — Pick a depth model

A design expresses separation through **borders** or through **elevation**. Pick one. Mixing them is
what produces the "every card has a grey stroke *and* a drop shadow" look that reads as unresolved.

Most premium work chooses elevation. If you do, the rule is absolute: **cards get no stroke.**

### What replaces the border

Three layers, together:

1. **A fill** — from the surface ramp (quiet / soft / featured)
2. **An inner top highlight** — a 1px inset white line. This is the light-catch that makes a surface
   feel like a physical plate rather than a colored rectangle.
3. **A two-part shadow** — contact + ambient.

```css
.surface {
  position: relative;
  isolation: isolate;
  overflow: hidden;
  border: none;                          /* the whole point */
  border-radius: var(--radius-plate);
  background:
    linear-gradient(165deg, rgba(255,255,255,.95) 0%, transparent 42%),  /* sheen */
    var(--surface-soft);
  box-shadow:
    var(--shadow-inset),                 /* inner top highlight */
    0 0 0 1px rgba(20,27,38,.05),        /* hairline *as shadow*, not border */
    var(--shadow-2);
  backdrop-filter: blur(16px) saturate(140%);
  -webkit-backdrop-filter: blur(16px) saturate(140%);
}
```

The `0 0 0 1px` spread shadow is a definition edge that does not participate in layout, does not
inherit border-radius bugs, and can be animated. Not a border.

---

## Part 2 — The shadow ladder

```css
--shadow-1: 0 1px 2px rgba(23,37,84,.10), 0 8px  20px -12px rgba(23,37,84,.16);
--shadow-2: 0 2px 4px rgba(23,37,84,.10), 0 16px 36px -18px rgba(23,37,84,.20);
--shadow-3: 0 4px 8px rgba(23,37,84,.12), 0 28px 56px -20px rgba(23,37,84,.26);
```

Three properties make these read as real:

- **Two parts.** A tight contact shadow *grounds* the element; a wide soft ambient one *lifts* it.
  One shadow can do one or the other, never both — that is why single-shadow cards look like stickers.
- **Brand-tinted, never neutral black.** `rgba(23,37,84,…)` is the navy family. A `rgba(0,0,0,…)`
  shadow on a colored ground is the clearest sign the shadow came from a framework default.
- **Negative spread on the ambient part.** `-18px` pulls the blur back in so the shadow does not
  bleed into neighbours.

Map the steps to meaning, not to taste: `--shadow-1` quiet/secondary, `--shadow-2` default card,
`--shadow-3` featured or hovered.

---

## Part 3 — Interaction moves up the ladder

Hover must change **elevation**, not just color. A real lift plus the next shadow tier:

```css
.surface-interactive {
  transition:
    transform  var(--dur-micro) var(--ease),
    background var(--dur-ui)    var(--ease),
    box-shadow var(--dur-ui)    var(--ease);
}

.surface-interactive:hover,
.surface-interactive:focus-visible {
  background: linear-gradient(165deg, rgba(52,98,183,.10) 0%, transparent 50%), #fff;
  box-shadow:
    inset 0 1px 0 rgba(255,255,255,.9),
    0 0 0 1px rgba(52,98,183,.16),      /* edge picks up the accent */
    var(--shadow-3);                     /* ← up a tier */
  transform: translate3d(0, -3px, 0);    /* a real rise sells the change */
}

.surface-interactive:active {
  transform: scale(0.995);               /* tactile press */
}
```

`-3px` is deliberate. `-1px` is invisible; `-8px` is a trampoline. Note `:focus-visible` gets the
same treatment as `:hover` — keyboard users get the same affordance.

### Buttons

Same principle, plus a composable transform so effects do not overwrite each other:

```css
.btn {
  --mag-x: 0px;  --mag-y: 0px;           /* magnetic pull, set by JS on fine pointers only */
  min-height: 2.75rem;                    /* touch target floor */
  border-radius: 9999px;
  background: var(--accent-600);          /* darker step so white text clears AA */
  color: var(--btn-text);
  letter-spacing: 0.06em;
  text-transform: uppercase;
  box-shadow: var(--shadow-btn);
  transform: translate3d(var(--mag-x), var(--mag-y), 0);
  transition: transform var(--dur-micro) var(--ease),
              background var(--dur-ui) var(--ease),
              box-shadow var(--dur-ui) var(--ease);
}
.btn:active {
  transform: translate3d(var(--mag-x), var(--mag-y), 0) scale(0.97);  /* composes, not replaces */
  background: var(--accent-700);
}
```

Writing `transform: scale(0.97)` in `:active` would silently discard the magnetic offset. Composing
through custom properties is the fix.

---

## Part 4 — Where borders *are* allowed

Keep an explicit short list in `DESIGN.md`. Typically:

- Ghost/secondary buttons (a stroke is the whole design)
- Small control circles and icon buttons
- Focus rings
- Single hairline rules joining sections (`--hairline`, 1px, as a `background`, not a `border`)

Never: full card strokes, input wrappers that duplicate the card stroke, nested bordered boxes.

---

## Part 5 — Context inversion (the pass nobody does)

**The failure.** A fixed header styled for a light page scrolls over a dark anchor band. Its links,
brand mark, ghost button, and focus rings were all colored for light ground. They now sit at 2–3:1.
The source project measured a real **2.37:1** failure here before the pass.

This affects: nav, sticky mobile CTA, floating action buttons, scroll progress indicators, back-to-top,
cookie banners — anything `position: fixed`.

### Detect the band

Use `IntersectionObserver` on sections marked as dark, and stamp a data attribute on the chrome.
Never hardcode scroll offsets — they break the moment content length changes.

```tsx
// Sections opt in:  <section data-nav-dark> … </section>
useEffect(() => {
  const darkBands = document.querySelectorAll("[data-nav-dark]");
  const io = new IntersectionObserver(
    (entries) => {
      const overlapping = entries.some((e) => e.isIntersecting);
      headerRef.current?.toggleAttribute("data-over-dark", overlapping);
    },
    // A sliver at the top of the viewport = exactly the strip the header occupies.
    { rootMargin: "-1px 0px -99% 0px", threshold: 0 },
  );
  darkBands.forEach((b) => io.observe(b));
  return () => io.disconnect();
}, []);
```

### Invert everything under the scope

Two scopes: `.on-navy` for content *inside* a dark band, `[data-over-dark]` for chrome *crossing* one.

```css
/* Type roles — light-theme colors would vanish */
.on-navy .h1, .on-navy .h2, .on-navy .h3 { color: #ffffff; }
.on-navy .body1   { color: #c3d2ea; }                    /* 7.4:1 on the band */
.on-navy .eyebrow { color: #c3d2ea; }
.on-navy .divider { background: rgba(195,210,234,.2); }

/* CTAs invert: white fill separates from the dark field */
.on-navy .btn        { background: #fff; color: var(--accent-700); }
.on-navy .btn:hover  { background: #eef4fb; }
.on-navy .btn-ghost  { color: #c3d2ea; border-color: rgba(195,210,234,.36); }

/* Brand mark — saturated brand hues routinely fail on dark */
.on-navy .brand { color: var(--brand-mark-light); }      /* measure both variants on the band */

/* ── The one everyone forgets ── */
.on-navy :focus-visible,
[data-over-dark] a:focus-visible,
[data-over-dark] button:not(:disabled):focus-visible {
  outline-color: #9db4de;                                /* ~4.9:1 on the band */
}
```

### Inversion checklist

Scroll the page top to bottom, slowly, once. At every band crossing verify:

- [ ] Nav links legible (≥4.5:1)
- [ ] Brand/wordmark legible — check the *brand* color specifically, it fails most often
- [ ] Ghost/outline button border and label both visible
- [ ] Primary CTA still separates from the ground
- [ ] **Tab through it** — focus rings visible on the dark band
- [ ] Sticky mobile CTA, scroll progress, any other fixed chrome
- [ ] Nav glass/tint background works over both grounds

---

## Part 6 — Fallbacks

Backdrop blur is a progressive enhancement. Some users switch it off; honor that:

```css
@media (prefers-reduced-transparency: reduce) {
  .surface, .surface--featured, .surface--quiet {
    backdrop-filter: none;
    -webkit-backdrop-filter: none;
    background: #ffffff;      /* opaque fallback — contrast must still hold */
  }
}
```

Scrolled-nav glass needs the same treatment, and its tint should be the **page hue**, not neutral grey:

```css
.nav-scrolled {
  background: rgba(214, 226, 243, .72);   /* paper tint, same hue as the canvas */
  backdrop-filter: blur(14px) saturate(140%);
  box-shadow: 0 1px 0 var(--hairline), var(--shadow-1);
}
```
