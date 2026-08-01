# The Token Contract

One `:root` block owns every color, size, shadow, duration and easing on the site.
A raw hex, px value, or `cubic-bezier` inside a component is a defect.

---

## Rule 1 — name by role, never by value

```css
/* wrong — the name encodes the current answer */
--gray-500: #8d9fb9;
--blue-600: #3462b7;

/* right — the name encodes the job */
--text-tertiary: #8d9fb9;
--text-accent:   #3462b7;
```

Role names survive a palette change. Value names do not, and you end up with `--blue-600: #c8102e`,
which is how design systems die.

### The palette-pivot technique

The source project pivoted from a dark/ember theme to a light/navy one **without touching a single
component**, by repointing values behind unchanged names and documenting the indirection:

```css
/* Variable NAMES are kept to minimise churn across the codebase;
   their VALUES are repointed to the new palette:
     oz-black  → light page canvas (warm paper)
     oz-white  → primary ink (deep navy-black)
     oz-orange → navy accent (the sole accent, replaces ember)  */
--oz-black:  #d6e2f3;
--oz-white:  #141a29;
--oz-orange: #3462b7;
```

This is legitimate and pragmatic mid-project. Two conditions: the comment explaining the inversion is
mandatory, and you rename properly at the next quiet moment. Do not *start* a project this way.

---

## Rule 2 — measure contrast, and write the number down

Every text role carries its measured ratio in a comment. If the role appears on more than one ground,
annotate the worst case.

```css
--text-primary:   #141a29;  /* 14.6:1 on paper */
--text-secondary: #404c64;  /*  7.2:1 */
--text-tertiary:  #4f5f7a;  /*  5.0:1 on paper, 4.6:1 on the lighter band — clears AA on both */
--text-accent:    #3462b7;  /*  4.9:1 */
```

This is the highest-signal habit in the skill. It forces measurement instead of eyeballing, and a
later regression becomes visible in a diff.

Targets: **4.5:1** body text, **3:1** large text (≥24px or ≥19px bold) and UI boundaries.
Check every ground the role can land on — page canvas, tinted band, dark anchor band, card fill.

---

## Rule 3 — exactly one interactive accent

Everything clickable, focused, in-progress, or selected uses the one accent. Give it a small scale so
fill / hover / pressed are related, not arbitrary:

```css
--accent-500: #3462b7;  /* default: links, focus rings, kickers */
--accent-600: #264a97;  /* CTA fill — darker so white text clears AA */
--accent-700: #1f3775;  /* pressed */
--accent-hover: #2662ed; /* the electric spark — one step brighter, not a different hue */
--btn-text: #fbfdff;
```

A brand color that is **not** the accent (a logo hue you are obliged to use) gets its own token and is
scoped to the wordmark. Give it a light variant for dark bands, because saturated brand colors
routinely fail contrast on a dark field:

```css
--brand-mark:       #c8102e;  /* wordmark only */
--brand-mark-light: #ff8a99;  /* dark bands — measure both; the base will usually fail there */
```

---

## Rule 4 — the full contract

```css
:root {
  /* Ground + ink */
  --canvas:  #d6e2f3;
  --ink:     #141a29;
  --hairline: rgba(38, 74, 151, 0.2);

  /* Text roles — three levels, ratios annotated */
  --text-primary:   #141a29;  /* 14.6:1 */
  --text-secondary: #404c64;  /*  7.2:1 */
  --text-tertiary:  #4f5f7a;  /*  5.0:1 */
  --text-accent:    #3462b7;  /*  4.9:1 */

  /* Accent scale + brand (above) */

  /* Surface ramp — quiet / default / featured */
  --surface-quiet:    rgba(255,255,255,0.72);
  --surface-soft:     rgba(255,255,255,0.90);
  --surface-featured: #ffffff;
  --surface-inset:    rgba(20,27,38,0.04);

  /* Depth ladder — two-part, brand-tinted (see surfaces-depth.md) */
  --shadow-1: 0 1px 2px rgba(23,37,84,.10), 0 8px  20px -12px rgba(23,37,84,.16);
  --shadow-2: 0 2px 4px rgba(23,37,84,.10), 0 16px 36px -18px rgba(23,37,84,.20);
  --shadow-3: 0 4px 8px rgba(23,37,84,.12), 0 28px 56px -20px rgba(23,37,84,.26);
  --shadow-inset: inset 0 1px 0 rgba(255,255,255,.9);

  /* Motion — one curve, three durations */
  --ease:      cubic-bezier(0.22, 1, 0.36, 1);
  --dur-micro: 150ms;
  --dur-ui:    220ms;
  --dur-enter: 300ms;

  /* Tracking roles */
  --track-label:   0.08em;
  --track-eyebrow: 0.2em;

  /* Type families by role */
  --font-display: var(--font-space-grotesk), system-ui, sans-serif;
  --font-body:    var(--font-plus-jakarta), system-ui, sans-serif;

  /* Layout + type scale — see typography.md */
  --site-padding-x: 3.125vw;
  --radius-plate: 1rem;
}
```

---

## Rule 5 — spacing is a mental model, not a list

Commit to `4 / 8 / 16 / 24 / 32 / 48 / 64` and use nothing else. You do not need tokens for these if
your utility framework already exposes the scale — you need the **discipline** of never writing `13px`
or `mt-[27px]`. One arbitrary value invites the next.

---

## Auditing the token layer

```bash
# raw hex outside the token file
rg '#[0-9a-fA-F]{3,8}\b' src/ --glob '!**/globals.css'

# hardcoded durations and curves in components
rg 'cubic-bezier|[0-9]+ms\b' src/components/

# arbitrary spacing escapes
rg '\[[0-9]+px\]' src/
```

Each hit is either a defect or a token that does not exist yet.
