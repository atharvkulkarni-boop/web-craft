# Typography

Timid type is the slop tell that survives every other fix. A page can have a perfect palette, real
shadows, and honest copy, and still read as generated because the headline is 40px.

---

## The scale: one unit, one ramp

Derive every size from a single viewport-linked unit so the whole ramp scales together, instead of
re-guessing sizes at each breakpoint.

```css
:root {
  --site-padding-x: 3.125vw;
  --inner-width: calc(100vw - var(--site-padding-x) * 2);

  /* One unit: tracks width, but capped by height so a short wide window
     doesn't produce a headline taller than the viewport. */
  --screen-unit: min(
    var(--inner-width) / (1920 - 60 * 2),
    100vh / 1024 * 1.25
  );

  --h1:    clamp(3rem,    calc(104 * var(--screen-unit)), 6.5rem);
  --h2:    clamp(2rem,    calc(60  * var(--screen-unit)), 4.25rem);
  --h3:    clamp(1.35rem, calc(34  * var(--screen-unit)), 2.125rem);
  --body1: clamp(1.05rem, calc(20  * var(--screen-unit)), 1.3125rem);
  --sub1:  clamp(0.75rem, calc(13  * var(--screen-unit)), 0.9375rem);

  /* The gasp moment — oversized editorial numeral */
  --index: clamp(3.25rem, calc(96 * var(--screen-unit)), 6rem);
}
```

The `104`, `60`, `34`, `20` are the design sizes at the 1920px reference width. Read the ramp as a
sentence: *"104 down to 20"* is a **5.2:1 display-to-body ratio**. That is the number that matters.

### Ratios to hit

| Relationship | Target | Why |
| --- | --- | --- |
| Display → body | **≥ 5:1** | Below 3:1 the page reads as a document |
| Gasp numeral → body | **~10:1** | One per page. This is what gets remembered |
| h2 → h3 | ~1.7–2.0 | A larger gap leaves h3 orphaned as "big body text" |

The h2→h3 step is the one that goes wrong. In the source build `--h3` was raised specifically to
"bridge the h2→h3 gap" after a review found card titles reading as body copy.

### Mobile override

Override only the tokens, never the components. Re-anchor the unit to the mobile reference width:

```css
@media (max-width: 767.98px) {
  :root {
    --site-padding-x: calc(16 * 100vw / 375);
    --screen-unit: calc(var(--inner-width) / (375 - 16 * 2));
    --h1:    clamp(2.15rem, calc(46 * var(--screen-unit)), 2.7rem);
    --h2:    clamp(1.75rem, calc(46 * var(--screen-unit)), 2.4rem);
    --index: clamp(2.75rem, calc(56 * var(--screen-unit)), 4rem);
  }
}
```

Note `--h1` and `--h2` converge on mobile — at 375px there is no room for a 5:1 ramp, and pretending
otherwise produces a headline that wraps six times.

---

## Role classes, not utility soup

Every piece of text routes through one named role. Components never set `font-size` directly.

```css
.h1 {
  font-family: var(--font-display);
  font-size: var(--h1);
  font-weight: 500;
  line-height: 0.98;          /* sub-1 leading is what makes display type feel set */
  letter-spacing: -0.04em;
  text-transform: uppercase;
  text-wrap: balance;
  color: var(--text-primary);
}

.h3 {
  font-family: var(--font-display);
  font-size: var(--h3);
  font-weight: 500;
  line-height: 1.12;
  letter-spacing: -0.005em;   /* near-zero: negative tracking crowds uppercase caps at this size */
  text-transform: uppercase;
  text-wrap: balance;
}

.body1 {
  font-size: var(--body1);
  line-height: 1.65;
  color: var(--text-secondary);
  max-width: 56ch;            /* measure is a rule, not a suggestion */
  text-wrap: pretty;
}

.eyebrow {                    /* wide-tracked micro-label */
  font-size: 0.75rem;
  font-weight: 600;
  letter-spacing: var(--track-eyebrow);
  text-transform: uppercase;
  color: var(--text-accent);  /* spreads the accent hue through every section */
}

.stat {
  font-family: var(--font-display);
  font-variant-numeric: tabular-nums lining-nums;  /* numbers must not jitter */
  letter-spacing: -0.03em;
}
```

### The tracking rule

**Letter-spacing is a function of size and casing, never a constant.**

- Large display, mixed or upper: `-0.03em` to `-0.05em`. Big type has too much natural space.
- Mid-size uppercase (card titles, ~20–34px): `≈ 0` to `-0.005em`. Negative tracking crowds caps here.
- Small uppercase labels: `+0.08em` to `+0.2em`. Small caps need air to stay legible.

Applying one tracking value across the ramp is a direct tell.

---

## The gasp moment

Give the page exactly one oversized typographic element — a step numeral, a year, a headline metric.
Color it as **structure, not content**: a low-alpha accent so it reads as architecture behind the
copy rather than shouting.

```css
.index {
  display: block;
  font-family: var(--font-display);
  font-size: var(--index);
  line-height: 0.82;
  letter-spacing: -0.05em;
  font-variant-numeric: tabular-nums lining-nums;
  color: rgba(52, 98, 183, 0.38);   /* accent at low alpha — structural, not noisy */
}
```

One per page. Two is a motif; three is wallpaper.

---

## Family selection

- **Two families maximum**: a display face for headings, a humanist sans for body and UI.
  One family used at genuinely different weights and sizes also works and is often better.
- **Do not ship Inter unmodified.** It is the single most recognisable AI-default typeface on the web.
  If you use it, pair it with a distinctive display face carrying all the character.
- Load with `font-display: swap` and preload only the weights actually used. Two weights is usually
  the honest answer; five is a performance bill for nothing.
- Variable fonts if you genuinely need intermediate weights or optical sizing — otherwise static
  subsets are smaller.

## Checks

- [ ] Display-to-body ≥ 5:1
- [ ] Exactly one ~10:1 gasp element
- [ ] h2→h3 step ≤ 2.0
- [ ] Tracking varies by size and casing
- [ ] Body has an explicit measure (`max-width` in `ch`)
- [ ] Numerals are tabular anywhere they change or align
- [ ] `text-wrap: balance` on headings, `pretty` on body
- [ ] No component sets `font-size` directly
