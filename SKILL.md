---
name: web-craft
description: 'Build discipline for shipping web UI that reads as designed, not generated. Distilled from a production site build. Use when the user says "make it look designed", "anti-AI-slop", "it looks like AI made it", "refined", "polished", "premium feel", "craft pass", "design system for this site", "audit my UI for slop", or when starting a marketing/landing/brand site that must not look templated. Enforces a written design contract with an anti-pattern list, a role-based token system with measured contrast, borderless elevation, context inversion over dark bands, motion tokens, content-honesty rules, and a subtraction pass. For aesthetic direction see top-design; for visual fundamentals see refactoring-ui; for brief-driven style inference see design-taste-frontend. This skill is the process that holds those together.'
metadata:
  author: athar
  version: "1.0.0"
  source: distilled from a shipped production build
---

# web-craft

> The difference between "AI made this" and "someone made this" is almost never talent.
> It is that a person made **decisions and wrote them down**, then did passes the generator never does.

This skill is a **process**, not a style. It does not tell you to use navy, or uppercase display type,
or a marquee. It tells you what has to be *decided*, *recorded*, and *checked* so the output holds
together. Every code sample here is illustrative of a pattern, not a palette to copy.

---

## When this fires

Use it when the deliverable is a real site or page whose quality is the point — marketing, landing,
brand, portfolio, client work. Use it on redesigns and polish passes too ("this looks like slop, fix it").

Do **not** use it for internal dashboards, admin CRUD, data tables, or docs sites. Those want
consistency and density, not craft theatre. Different job.

---

## The core claim

AI-generated UI fails in a specific, diagnosable way. It is not ugly. It is **undecided**.
Every value is a local guess, so nothing accumulates into a system:

| Slop tell | What it actually means |
| --- | --- |
| Every card has `border: 1px solid #e5e7eb` | No depth model was chosen, so a border stands in for one |
| Three equal feature cards | No hierarchy was decided, so everything is peer-level |
| Type sizes are 16/18/24/32/48 | The scale is the framework's default, not a decision |
| One flat `opacity` fade on scroll | Motion was added, not designed |
| Text is `#666` on `#fff` everywhere | Contrast was never measured, only eyeballed |
| Nav links vanish over the dark footer | No one scrolled the whole page once |
| "Scroll to explore" / "Elevate your workflow" | Copy is filler because the facts were never gathered |
| Gradient purple hero, floating glass card | The one visual idea in the training distribution |

The fix for each is the same shape: **decide it once, name it, write it down, apply it everywhere.**

---

## Workflow

Run these in order. Do not skip 1 — everything downstream references it.

### 1. Write the design contract *before* building

Create `DESIGN.md` at the repo root. It is short, opinionated, and binding. Sections:

1. **Visual theme & atmosphere** — one paragraph, concrete nouns. Not "modern and clean".
2. **Color palette & roles** — every color with the *role* it plays, and a **Banned** line.
3. **Typography rules** — families, weights, casing, and a **Banned** line.
4. **Layout principles** — what the page shape is, and what it refuses to be.
5. **Motion** — what moves, driven by what, and what happens under reduced motion.
6. **Anti-patterns** — the explicit no-list for *this* project.

The Banned/Anti-pattern lines are the load-bearing part. A contract that only says yes constrains nothing.

> The register to aim for:
> *"Lit workbench. Cold-rolled steel is the continuous world — hero and body sections share one
> atmosphere. The bike is a dark object under a single overhead lamp, not a floating card."*
> — followed by *"Banned: pure black, cool blue, multi-accent rainbow UI. Brass stays on the
> wordmark; signal red is the only interactive accent."*

Ambiguity here becomes drift later. If the user has not given you enough to write section 2 or 3,
**ask** — this is one of the few places worth blocking on.

→ Full template and worked example: `references/design-contract.md`

### 2. Build the token layer, roles first

One `:root` block is the single source of truth. Rules:

- **Name tokens by role, never by value.** `--text-tertiary`, not `--gray-500`. Roles survive a
  palette pivot; values do not. (In the source project the entire palette flipped from dark/ember to
  light/navy by repointing values behind unchanged names — zero component churn.)
- **Annotate every text role with its measured contrast ratio, in a comment.** This is the single
  highest-signal anti-slop habit in the whole skill. It forces you to actually measure, and it makes
  a later regression visible in review.
- **Exactly one interactive accent.** Every clickable, focused, or in-progress thing uses it.
  A brand color that is *not* the accent (a logo hue) gets its own token and is scoped to the wordmark only.
- **Three text levels, not five.** primary / secondary / tertiary. If you need a fourth, you have a
  layout problem, not a color problem.

```css
--text-primary:   #141a29;  /* 14.6:1 on paper */
--text-secondary: #404c64;  /*  7.2:1 */
--text-tertiary:  #4f5f7a;  /*  5.0:1 on paper, 4.6:1 on the tinted band — clears AA on both */
--text-accent:    #3462b7;  /*  4.9:1 — sole interactive accent */
```

→ Full token contract: `references/tokens.md`

### 3. Type: one scale, real contrast, one gasp

- Derive sizes from a single viewport unit + `clamp()`, so the whole ramp scales together instead of
  being re-guessed per breakpoint.
- **Display-to-body ratio should be ~5:1 or more.** Timid type is the most common slop tell that
  survives every other fix. If your h1 is 2.5× body, it is a document, not a design.
- Give the page **one oversized typographic moment** — a numeral, a year, a metric — at ~10:1,
  colored as structure (low-alpha accent) rather than as content. This is what people remember.
- Set `line-height` and `letter-spacing` per role. Negative tracking that flatters a 100px display
  heading crowds a 20px uppercase card title — the source project shipped `-0.04em` on h1 and
  `-0.005em` on h3 for exactly this reason.

→ Scale construction, role classes, and the mobile override: `references/typography.md`

### 4. Depth: pick a model, then delete the borders

Decide *once* whether the design uses borders or elevation. Then be absolute about it.

If elevation: cards get **no stroke**. They get a fill, an inner top highlight, and a **two-part
shadow** — a tight contact shadow plus a soft wide ambient one. Tint both with the brand hue; a
neutral-black shadow on a colored ground is the giveaway that the shadow came from a default.

```css
--shadow-2:
  0 2px  4px      rgba(23,37,84,.10),   /* contact — grounds it   */
  0 16px 36px -18px rgba(23,37,84,.20); /* ambient — lifts it     */
--shadow-inset: inset 0 1px 0 rgba(255,255,255,.9);
```

Three elevation steps, mapped to meaning (quiet / default / featured). Hover moves a card **up the
ladder** — a real `translate3d(0,-3px,0)` plus the next shadow tier, not a color nudge.

→ Surface recipes, hover/active states, transparency fallback: `references/surfaces-depth.md`

### 5. Motion: tokens, transform-only, and an off switch

Three durations and one curve, as tokens. Micro (~150ms) for press and hover, UI (~220ms) for state,
enter (~300ms) for reveals. Animate `transform` and `opacity` only.

One shared reveal for the whole site. Make it a **rise plus a clearing blur**, not a flat fade —
same cost, reads as intent:

```js
initial: { opacity: 0, y: 16, filter: "blur(6px)" }
animate: { opacity: 1, y: 0,  filter: "blur(0px)" }
```

Any looping animation must pause on hover, on `focus-within`, on touch-hold, and when off-screen.
`prefers-reduced-motion` kills it outright. This is WCAG 2.2.2, and it is also just correct.

→ Marquee implementation, scroll-scrub, reduced-motion strategy: `references/motion.md`

### 6. The inversion pass ← *the one nobody does*

Scroll the entire page slowly, once, and watch the **persistent chrome** — nav, sticky CTA, focus
rings — as it crosses every band.

When fixed chrome passes over a dark section, its light-theme colors either vanish or fail contrast.
The source project measured a real 2.37:1 failure here. The fix is a scoped inversion: mark the dark
band, and re-color type, buttons, brand, *and focus outlines* under that scope.

```css
.on-navy .oz-h2        { color: #fff; }
.on-navy .oz-body1     { color: #c3d2ea; }   /* 7.4:1 on the band */
.on-navy .btn-oz       { background: #fff; color: var(--accent-700); }
.on-navy :focus-visible{ outline-color: #9db4de; }  /* ← the one everyone forgets */
```

Detect the band from JS (`IntersectionObserver` → `data-over-dark` on the header) rather than
hardcoding scroll offsets.

→ Detection pattern and full inversion checklist: `references/surfaces-depth.md`

### 7. Content honesty

Slop copy comes from having no facts. Fix the input, not the prose.

- Put **business facts in one module** (`lib/site.ts`), separate from all design chrome. Hours,
  numbers, addresses, links.
- Compute live state from real data — opening hours resolved in the business's actual timezone,
  not a hardcoded "Open now" badge.
- **Never invent** reviews, testimonials, client logos, metrics, guarantees, or warranty terms.
  If the user has not supplied them, ship the section with real content or do not ship the section.
- Ban filler: "Scroll to explore", "Elevate your…", "Seamlessly…", "In today's fast-paced world".
- Deep-link actions with context — a WhatsApp CTA should prefill the model and issue fields, not
  open an empty thread.

→ Facts-module pattern and the banned-copy list: `references/content.md`

### 8. The subtraction pass

Before shipping, find the one element that is decoration rather than structure, and **delete it**.
The source project cut a full-perimeter animated edge wave late in the build — it was technically
impressive and it read as noise. The commit comment recording *why* is worth as much as the deletion.

Then run the audit.

---

## The slop audit

Run this on your own output before showing the user. Any **no** is a fix, not a note.

**Squint test** — blur the page hard. Do three levels of hierarchy survive? If it flattens to one
grey mass, the type ramp is too timid or everything is at one elevation.

- [ ] Is there a `DESIGN.md` with a populated Anti-Patterns section?
- [ ] Is every color, size, shadow and duration a **role-named token**? Any raw hex in a component is a defect.
- [ ] Does every text role carry a measured contrast ratio in a comment, including on tinted/dark bands?
- [ ] Exactly one interactive accent across the whole site?
- [ ] Display-to-body size ratio ≥ 5:1, and exactly one ~10:1 moment?
- [ ] Zero `1px solid` card strokes (if the design chose elevation)?
- [ ] Are shadows two-part and brand-tinted?
- [ ] Does hover move cards up a real elevation step (transform + next shadow tier)?
- [ ] Did you scroll the full page and check nav, sticky CTA, **and focus rings** over every band?
- [ ] Does `:focus-visible` land visibly on every interactive element, in both normal and inverted contexts?
- [ ] Do looping animations pause on hover / focus-within / touch / off-screen?
- [ ] Does `prefers-reduced-motion` actually stop things? `prefers-reduced-transparency` fall back cleanly?
- [ ] `env(safe-area-inset-*)` respected on any fixed bottom chrome?
- [ ] Are all facts real and sourced? Zero invented reviews, stats, or guarantees?
- [ ] Is the layout full-bleed-and-asymmetric, or did it default to a boxed max-width with three equal cards?
- [ ] What did you **delete** in the subtraction pass?

If you cannot answer the last one, you have not finished.

---

## References

| File | Contents |
| --- | --- |
| `references/design-contract.md` | `DESIGN.md` template + worked example + anti-pattern library |
| `references/tokens.md` | Full `:root` contract, role naming, palette-pivot technique |
| `references/typography.md` | `--screen-unit` scale, role classes, mobile overrides |
| `references/surfaces-depth.md` | Borderless elevation, shadow ladder, context inversion |
| `references/motion.md` | Motion tokens, reveals, accessible marquee, reduced motion |
| `references/content.md` | Facts module, live state, banned copy, honest sections |
