# The Design Contract

`DESIGN.md`, at the repo root, written **before** the first component. It is the artifact that makes
the difference between a site that was decided and a site that was generated.

Keep it under one screen. It is a contract, not documentation. If it grows past ~60 lines, you are
describing implementation instead of committing to decisions.

---

## Template

```markdown
# Design System: <Project>

## 1. Visual Theme & Atmosphere
<One paragraph. Concrete nouns and a physical metaphor. State what the page *is*, then what the
subject *is not*. Optionally end with numeric dials: Density N/10, Variance N/10, Motion N/10.>

## 2. Color Palette & Roles
- **<Name>** (#hex) — <the single role it plays>
- ...
- **Banned:** <colors and combinations that are out, and why>

## 3. Typography Rules
- **Display:** <family, casing, weight, tracking, scale behavior>
- **Body:** <family, weight, line-height, max measure>
- **Labels:** <casing, tracking, size role>
- **Banned:** <families and filler strings that are out>

## 4. <Signature> Architecture
<The one structural idea the page is built around — hero grammar, scroll spine, editorial grid.
Name it, describe its parts, state what it refuses to be.>

## 5. Component Stylings
- **Buttons / Surfaces / Nav / Mobile chrome** — one line each, naming the token or class that owns it
- **Borders allowed:** <the explicit short list>

## 6. Layout Principles
<Page shape. Full-bleed vs boxed. Symmetry policy. How sections join.>

## 7. Motion
<What moves, what drives it, what happens under reduced motion.>

## 8. Anti-Patterns
<The no-list for this project. Be specific and slightly petty. This section is the point.>
```

---

## Worked example

An illustrative contract for a fictional bicycle workshop — note how much is phrased as refusal:

> **Visual Theme.** Lit workbench. Cold-rolled steel (#E8E9E6) is the continuous world — hero and body
> sections share one atmosphere. The bike is a dark object under a single overhead lamp,
> **not a floating card**. Density 3, Variance 7, Motion 8.
>
> **Color roles.** Steel — full page canvas. Graphite — primary type. Signal red — *sole* accent: CTAs,
> progress, focus, links. Brass — brand wordmark ONLY. White — elevated cards (soft shadow + hairline,
> borderless). **Banned:** pure black, cool blue, multi-accent rainbow UI.
>
> **Typography.** Display: uppercase, track-tight, weight 500, massive clamp scale. Body: same family,
> weight 400, relaxed, max ~65ch. **Banned:** Inter, generic serif, filler "Scroll to explore".
>
> **Borders allowed:** ghost pills, control circles, focus rings — **never full card strokes**.
>
> **Layout.** Full-bleed sections; container padding only — no boxed max-width "SaaS page".
> Asymmetric hero; **no 3 equal feature cards**. Section joins share one void — no seams mid-page.
>
> **Anti-Patterns.** Floating video square disconnected from page · multi-pin scroll · "Scroll for more"
> cues · warranty claims · invented reviews.

Two things to copy from this:

1. **Color entries name a role, not a vibe.** "Sole accent: CTAs, progress, focus, links" is checkable.
   "Primary brand color" is not.
2. **Anti-patterns mix visual and ethical.** "Invented reviews" and "warranty claims" sit in the same
   list as "multi-pin scroll", because both are ways the output stops being trustworthy.

---

## Anti-pattern library

Draw from these when writing section 8. Pick the ones that actually threaten *this* project.

**Layout**
- Boxed `max-width: 1200px; margin: 0 auto` on every section ("SaaS page")
- Three (or four) equal feature cards in a row
- Perfectly symmetric hero: centered headline, centered sub, centered button pair
- Icon-in-a-circle-above-a-title-above-two-lines, repeated
- Visible seams where section backgrounds change color mid-page

**Visual**
- `1px solid #e5e7eb` on every card
- Purple→blue gradient anything
- Glassmorphism applied because it is available, not because there is depth to express
- Neutral-black shadows on a colored ground
- More than one interactive accent color
- Emoji as iconography

**Type**
- Framework-default scale (16/18/24/32/48)
- h1 under 3× body size
- Inter, unmodified, everywhere
- The same `letter-spacing` at 100px and 20px

**Motion**
- Flat `opacity` fade as the only reveal
- Animating `top`/`left`/`width` instead of `transform`
- Looping animation with no pause affordance
- Motion that ignores `prefers-reduced-motion`
- Multiple pinned scroll sections fighting each other

**Copy**
- "Scroll to explore", "Elevate your…", "Seamlessly…", "In today's fast-paced world"
- Invented testimonials, logos, metrics, guarantees
- Lorem ipsum shipped anywhere
- Generic CTA verbs ("Get Started") where a real action exists ("Book a service slot")

---

## Keeping it honest

The contract is only worth something if it is enforced. Two habits:

- When you break a rule, **amend the contract in the same change** and say why. A contract with silent
  exceptions is worse than none.
- When you delete something, **record the deletion and the reason** where the code used to be:

  ```css
  /* EdgeGlow removed in the design pass — the warm perimeter rim read as a heavy orange bar and
     had no analogue on the reference site. Its mount is gone from layout.tsx. */
  ```

  This is how the next person (or the next session) avoids re-adding it.
