# design-review

A Claude Code skill that replaces the wall-of-prose plan with a wireframe review you can mark up.

Instead of describing what it's about to build, Claude publishes a single self-contained HTML
artifact. Every proposal gets **two phone wireframes side by side — what the app does today, and
what's being proposed** — the reasoning behind it, and a feedback block with stance buttons and a
notes box. Your notes persist in the page, so you can read it on your phone across three sittings.
When you're done you hit **Save notes for Claude**, it exports a markdown file, you say "done", and
round 2 reworks what you flagged and banks what you decided.

**[See an example review →](https://jinkweon-debug.github.io/design-review-skill/examples/mise-review-round-1.html)**
(a round-1 review for a made-up recipe app — the stance buttons and notes are live)

## The loop

```
you: "let's plan the next phase"
  → Claude reads the code, publishes round 1 as an artifact
  → you read it whenever, pick Yes / Discuss more / Not now per item, write notes
  → hit "Save notes for Claude" → tell Claude "done"
  → round 2: a Decided ledger of everything you said Yes to,
             reworked wireframes for everything you wanted to discuss
  → when it concludes, the decisions get written into the repo's own docs
```

## Install

```bash
git clone https://github.com/jinkweon-debug/design-review-skill.git
cp -r design-review-skill/skills/design-review ~/.claude/skills/
```

Then start a planning conversation and say "make me a design review for X", or just ask to plan a
feature — the skill's description triggers it on its own when the work has screens in it.

Project-scoped instead of global: copy it to `.claude/skills/` inside the repo you're planning.

## What's in here

| Path | What it is |
| --- | --- |
| `skills/design-review/SKILL.md` | The skill. How to research the current state, structure a section, run the feedback loop, and write decisions back into the repo. |
| `skills/design-review/assets/template.html` | The page scaffold — phone-frame CSS, the stance/notes script, the exporter, and a commented example section. |
| `examples/mise-review-round-1.html` | The example review — [served here](https://jinkweon-debug.github.io/design-review-skill/examples/mise-review-round-1.html) by GitHub Pages. |

## Things worth knowing before you use it

**The phone frames are phone frames.** This was built for an iOS app. The frame is a fixed-size
div with a `.screen` inside it and everything absolutely positioned — widening it for desktop is a
CSS edit, but out of the box every wireframe is 248×520.

**The mock helper classes are examples, not API.** `.vid`, `.scrub`, `.savesheet`, `.jcard` and
friends in the template are the mock primitives my own app needed (video surfaces, journal cards,
save sheets). Keep the ones that fit, delete the rest, add your own — the skill expects you to
write bespoke mock CSS per review and just asks you to keep it in the same visual register.

**Swap the palette.** The `:root` block at the top of the template is a neutral ink-and-paper
theme. If your project has its own colors, put them there; the skill treats the project's design
system as binding on every wireframe it draws.

**The exporter needs the `downloads` capability**, declared at publish time. If it isn't available
the page falls back to clipboard copy — the template handles both paths.

**It costs real tokens.** A round with a dozen sections is not a cheap turn. The trade is against
building the wrong screen and finding out after.

**The failure mode to watch:** wireframes of the *current* state drawn from the roadmap file
instead of from the code. Roadmaps go stale — a phase marked "queued" may have shipped — and a
review that misdescribes today costs you trust in every proposal on the page. The skill is
insistent about this and it is still the thing most worth checking.

## Related, and how this differs

- **`artifact-design` / `frontend-design`** ship built into Claude Code and cover making artifacts
  look good, including a wireframe playbook. Its critique pass is Claude reviewing its own plan
  before building — there's no artifact you mark up.
- **[Claude Design](https://www.anthropic.com/news/claude-design-anthropic-labs)** (Anthropic Labs)
  is the closest native thing: a canvas with inline comments and `/design-sync` back to Claude
  Code. It's a tool for making designs rather than for reviewing a plan against what your app does
  today, and per Anthropic's docs there's no approve/reject decision workflow.
- Community wireframe skills generate wireframes and stop; the review happens elsewhere.

What this adds is the **async round trip**: per-item stances that survive closing the tab, an
export back into the session, and a ledger that keeps decided things decided.

## License

MIT.
