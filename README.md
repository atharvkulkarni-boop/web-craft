# web-craft

A Claude Code / Claude Agent skill for shipping web UI that reads as **designed**, not generated.

> The difference between "AI made this" and "someone made this" is almost never talent.
> It is that a person made **decisions and wrote them down**, then did passes the generator never does.

This is a **process, not a style**. It ships no palette, no typeface, no signature layout. It tells you
what has to be decided, recorded, and checked so the output holds together — then gives you an audit to
run on your own work before you show anyone.

Distilled from a shipped production site rather than assembled from design blog posts. The failure
cases in here are real ones, including a measured 2.37:1 contrast failure on fixed nav chrome crossing
a dark band.

---

## The core claim

AI-generated UI fails in a specific, diagnosable way. It is not ugly — it is **undecided**. Every value
is a local guess, so nothing accumulates into a system.

| Slop tell | What it actually means |
| --- | --- |
| Every card has `border: 1px solid #e5e7eb` | No depth model was chosen, so a border stands in for one |
| Three equal feature cards | No hierarchy was decided, so everything is peer-level |
| Type sizes are 16/18/24/32/48 | The scale is the framework's default, not a decision |
| One flat `opacity` fade on scroll | Motion was added, not designed |
| Text is `#666` on `#fff` everywhere | Contrast was never measured, only eyeballed |
| Nav links vanish over the dark footer | No one scrolled the whole page once |
| "Scroll to explore" / "Elevate your workflow" | Copy is filler because the facts were never gathered |

The fix for each is the same shape: **decide it once, name it, write it down, apply it everywhere.**

---

## The workflow

1. **Write the design contract first** — `DESIGN.md` with a binding anti-pattern list. The no-list is
   the load-bearing part; a contract that only says yes constrains nothing.
2. **Build tokens by role, not value** — `--text-tertiary`, never `--gray-500`, with the measured
   contrast ratio in a comment beside every text role.
3. **Type: one scale, real contrast, one gasp** — ≥5:1 display-to-body, exactly one ~10:1 moment.
4. **Depth: pick a model, delete the borders** — two-part brand-tinted shadows, hover moves up a tier.
5. **Motion: tokens, transform-only, an off switch** — one curve, three durations, a real pause contract.
6. **The inversion pass** — fixed chrome crossing dark bands, *including focus rings*. Nobody does this one.
7. **Content honesty** — one facts module, live state computed in the right timezone, zero invented
   reviews or guarantees.
8. **The subtraction pass** — find the element that is decoration rather than structure, delete it,
   record why.

Then run the slop audit in `SKILL.md`. Any *no* is a fix, not a note.

---

## Contents

| File | Contents |
| --- | --- |
| `SKILL.md` | The workflow and the slop audit checklist |
| `references/design-contract.md` | `DESIGN.md` template, worked example, anti-pattern library |
| `references/tokens.md` | Full `:root` contract, role naming, palette-pivot technique |
| `references/typography.md` | Viewport-unit scale, role classes, the tracking rule |
| `references/surfaces-depth.md` | Borderless elevation, shadow ladder, context inversion |
| `references/motion.md` | Motion tokens, reveals, accessible marquee, reduced motion |
| `references/content.md` | Facts module, live state, banned copy, honest sections |

---

## Install

Clone into your skills directory:

```bash
git clone https://github.com/atharvkulkarni-boop/web-craft.git ~/.claude/skills/web-craft
```

Or keep it anywhere and symlink:

```bash
git clone https://github.com/atharvkulkarni-boop/web-craft.git ~/web-craft
ln -s ~/web-craft ~/.claude/skills/web-craft
```

Claude picks it up automatically. Invoke with `/web-craft`, or just describe the work — the skill
triggers on phrasing like *"make it look designed"*, *"it looks like AI made it"*, *"craft pass"*,
or *"audit my UI for slop"*.

---

## Scope

**Use it for** marketing sites, landing pages, brand sites, portfolios, client work — anything where
quality is the point. Works on redesigns and polish passes as well as new builds.

**Not for** internal dashboards, admin CRUD, data tables, or docs sites. Those want consistency and
density, not craft theatre. Different job.

**Companion skills.** This one is deliberately process-only, so it composes rather than competes. Pair
it with a skill that carries aesthetic direction (`top-design`), one that covers visual fundamentals
(`refactoring-ui`), or one that infers style from a brief. web-craft is the process that holds those
together.

---

## License

MIT — see [LICENSE](LICENSE).
