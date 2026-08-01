# Content Honesty

Slop copy is a symptom. Nobody writes "Elevate your workflow" because they think it is good — they
write it because they had no facts and needed the box filled. **Fix the input, not the prose.**

This is also the section with real stakes. Bad shadows are a taste problem. Invented reviews and
fabricated guarantees are a lie published under a real business's name, and the business carries the
consequence, not you.

---

## Rule 1 — one facts module, zero design chrome

Every business fact lives in one place, imported by components. Not scattered through JSX.

```ts
/**
 * Brand & business facts — <Business>
 * Kept through fresh rebuilds. No design chrome here.
 */
export const BRAND_SHORT = "Northgate Cycles";
export const PHONE = "+44 20 7946 0112";
export const ADDRESS = { line1: "12 Northgate Row", locality: "Clerkenwell", city: "London" };
export const HOURS = { monFri: "09:00–18:00", sat: "09:00–17:00", sun: null };
```

Two payoffs. A rebuild or redesign keeps every fact intact — you throw away the design, not the truth.
And there is exactly one place to check when the user says "the number changed".

---

## Rule 2 — compute live state, do not hardcode it

A hardcoded "Open now" badge is wrong most of the day. Resolve real state from real data, in the
business's **actual timezone** — not the server's, not the visitor's:

```ts
export function getOpenState(now = new Date()) {
  const parts = new Intl.DateTimeFormat("en-GB", {
    timeZone: "Europe/London",            // the business's zone, always explicit
    weekday: "short", hour: "2-digit", minute: "2-digit", hour12: false,
  }).formatToParts(now);

  const weekday = parts.find((p) => p.type === "weekday")?.value ?? "";
  const mins = Number(parts.find((p) => p.type === "hour")?.value ?? 0) * 60
             + Number(parts.find((p) => p.type === "minute")?.value ?? 0);

  if (weekday === "Sun") {
    return { open: false, label: "Closed today", detail: "Open Mon–Sat from 9:00 AM" };
  }
  const openMins = 9 * 60;
  const closeMins = weekday === "Sat" ? 17 * 60 : 18 * 60;
  // …
}
```

Note the closed-state copy still gives the user their next action ("Open Mon–Sat from 9:00 AM").
A dead end is a design failure even when the fact is correct.

Same principle elsewhere: derive "N years in business" from a founding year; derive review counts from
the review data; never freeze a number that will silently rot.

---

## Rule 3 — deep-link actions with context

A CTA that opens an empty message thread pushes the work back onto the user. Prefill it:

```ts
export function whatsappForIssue(issue: string) {
  return `https://wa.me/${NUMBER}?text=` + encodeURIComponent(
    `Hi ${BRAND_SHORT} - I need a quote for: ${issue}.\nBike model:\n`,
  );
}
```

Now the "Buckled wheel" card's button opens a thread that already says which repair, with a blank to
fill. Same idea for `mailto:` subjects, prefilled form params, and map links that carry the place name
rather than bare coordinates.

---

## Rule 4 — never invent these

Not as placeholders, not "for now", not with a TODO:

- Reviews, testimonials, quotes, names, star ratings
- Client or partner logos
- Statistics — repairs completed, years in business, satisfaction rates
- Guarantees, warranty terms, turnaround promises
- Certifications, awards, press mentions
- Prices

Each is a factual claim a real business becomes accountable for. Getting a warranty term wrong on a
service business's site is a consumer-protection problem, not a copy problem.

**When the content does not exist**, in order of preference:

1. Ask the user for the real content.
2. Ship the section wired to a real source (a reviews API, a CMS field) rendering empty until populated.
3. Cut the section. A page without a testimonial band is fine. A page with fake testimonials is not.
4. If the user explicitly wants placeholders for layout, mark them unmistakably — `[PLACEHOLDER —
   replace before launch]` — and list every one in your handoff message.

Never silently swap invented content in. If a section renders from a data file, say so plainly:
*"Reviews render from `lib/reviews.ts` — the 8 entries there are the ones you supplied; add more there."*

---

## Rule 5 — banned copy

Filler that signals nobody wrote it:

- **Scroll cues** — "Scroll to explore", "Scroll for more", "↓ Discover". If the page does not
  invite scrolling on its own, a label will not fix it.
- **Verb inflation** — "Elevate", "Unlock", "Supercharge", "Revolutionize", "Transform your…"
- **Empty openers** — "In today's fast-paced world", "We believe that…", "Welcome to our website"
- **Adverb padding** — "Seamlessly", "Effortlessly", "Simply put"
- **Generic CTAs** where a real action exists — "Get Started" → "Book a service slot";
  "Learn More" → "See what a drivetrain rebuild costs"
- **Hedged claims** that dodge the fact — "industry-leading", "trusted by many", "world-class"
- **Lorem ipsum**, anywhere, ever

The test: could this sentence appear on a competitor's site with no edits? If yes, it says nothing.

### Write from the specific

| Filler | Grounded |
| --- | --- |
| "Fast, reliable service" | "Most repairs done same day, while you wait" |
| "Expert technicians" | "Same two mechanics since 2011" |
| "Get Started" | "Send us your frame number on WhatsApp" |
| "Quality parts" | "Shimano and SRAM drivetrains, fitted in-house" |

Each right-hand version needs a fact. That is the point — it forces you to go get one.

---

## Rule 6 — sections earn their place

Every section answers a real question the visitor has. If you cannot name the question, cut it.

A page with four sections that each do work beats eight where half are structural filler
("Our Values", "Why Choose Us", "Our Process" with three vague steps).

Signs a section is filler: it could be dropped with nothing lost; its content is three synonyms for
"we are good"; it exists because the template had a slot there.

---

## Checks

- [ ] All business facts in one module, no design chrome in it
- [ ] Timezone-correct live state, no hardcoded "Open now"
- [ ] Derived numbers computed, not frozen
- [ ] CTAs deep-link with context prefilled
- [ ] Zero invented reviews, logos, stats, guarantees, certifications, prices
- [ ] Any placeholder is unmistakably marked *and* listed in the handoff message
- [ ] No banned phrases; no sentence that could appear unedited on a competitor's site
- [ ] Every section answers a nameable visitor question
- [ ] Closed/empty/error states still offer a next action
