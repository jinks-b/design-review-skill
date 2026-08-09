---
name: design-review
description: >
  Build an interactive design-review artifact during planning: phone wireframes
  (current vs. proposed), the reasoning behind each proposal, stance buttons and
  note boxes the user fills in at their own pace, and a "Save notes for Claude"
  exporter — then iterate in rounds from the notes they save. Use this whenever
  the user wants to review a plan, phase, or feature visually before building;
  asks for wireframes, mockups, or "something I can comment on"; says "make a
  review artifact" or "like the last review"; or comes back with a saved notes
  file (or says "done") from a previous round. Also use it for round 2+: reworking
  flagged items, answering the user's questions, and moving decided items to a
  ledger. Prefer this over plain-text plans whenever the work being planned has
  screens the user will look at. Also use it after reviewed work ships, to check
  the built screen against what the review decided.
---

# Design review artifacts

A design review artifact is a single self-contained HTML page, published with the
Artifact tool, that a user reads on their own time. It replaces a wall of prose
plan with something they can *see* and react to: every proposal gets wireframes,
the reasoning, and a feedback block. Their notes persist in the page itself
(localStorage) and export as a markdown file you read to start the next round.

The loop: **you publish round N → they read, pick stances, write notes → they hit
"Save notes for Claude" and tell you "done" → you read the notes file → round N+1
reworks what they flagged and banks what they decided.**

## Before you draw anything

1. **Read the actual code and docs of the thing being reviewed.** Every "today"
   wireframe must be drawn from what the app actually does right now — open the
   views, read the policies, check what shipped. Do not draw the current state
   from memory or from a roadmap file: roadmaps go stale (a phase marked "queued"
   may have shipped), and a review that misdescribes today forfeits the user's
   trust in the proposals. When docs and code disagree, the code wins.
2. **Collect the user's own words.** Every section that responds to something the
   user said quotes them verbatim in a `from-note` block. This is how they know
   you heard the actual concern, not a paraphrase of it.
3. **Honor the project's design system.** If the project has design rules
   (DESIGN.md, a palette, a copy covenant), the wireframes obey them — drawing a
   proposal that breaks the project's own rules is a bug. Put the project's
   non-negotiables in a "Guardrails" ledger at the top of round 1 and design
   inside them.
4. **Load the `artifact-design` skill before writing the page, and the
   `artifact-capabilities` skill before touching `window.claude.*`** — the
   exporter needs the `downloads` capability declared at publish time.

## Building the page

Start from `assets/template.html` in this skill — it carries the page scaffold,
the phone-frame CSS, the feedback/exporter JavaScript, and a commented example
section. Copy it to the session scratchpad, fill the placeholders
(`__TITLE__`, `__STORE_KEY__`, `__EXPORT_FILENAME__`, `__EXPORT_HEADING__`),
swap the `:root` palette tokens for the project's own, and write sections into
the marked content area.

### Section anatomy

Each reviewable item is one `<section class="theme">` containing one
`<article class="item">`:

- `theme-tag` — item id, size estimate, schema/migration impact, dependencies
  ("E2 · Size M–L · Additive schema · rides with E0"). Facts, not decoration.
- `h2` + `item-head`: an `h3` stating the claim, a `from-note` quoting the user
  (when the section answers them), a `take` paragraph leading with the verdict,
  then a `qa` list — each `li` opens with a bolded claim (`<span class="q">`)
  and explains the why. Flag any deviation from the project's existing decisions
  explicitly ("Deviation flag: …") rather than silently overriding them.
- One or two `pair` grids of phone frames. The strongest layout is
  **current vs. proposed**: label the left phone with what today truly looks
  like (`frame-label`, no "proposed" class) and the right with the proposal
  (`frame-label proposed`). Use `hl` (dashed outline) on the element that
  changed and `hl-note` for short callouts; `anno` under each phone carries one
  sentence of interpretation. Draw honest flaws in the "today" phone — the
  comparison is the argument.
- A `fb` feedback block: `data-item` key, empty `stances` div (buttons are
  injected by the script), a `textarea` with a placeholder that asks the one or
  two questions you most want answered, and a `saved-tick`.

Close the page with: an "Order & sizes" table (`fw`) giving the recommended
build order with a one-line "why this order" per row (it gets its own `fb`
block), and an "Anything else" catch-all `fb` in a `final` card that tells the
user exactly how to finish: Save notes → tell Claude "done".

### Wireframes

The template's phone frame is a fixed-size mock that auto-scales to its column.
Compose screens from absolutely-positioned divs with small font sizes — the
template includes helpers for dark screens (`vid`, `rail`, `cap`, `scrub`,
`meta-bl`) and paper screens (`phone lightui`, `jhead`, `jcard`, `jrow`,
`rcard`, `savesheet`). Add bespoke mock CSS per page as needed (calendars,
widgets, trays) — keep it in the same visual register. Mock data should be
plausible and *internally consistent*: dates carry the correct day of the week,
counts match what the mock visibly shows ("12 days told" over a grid showing 5
filled days will be noticed), and the same fictional entry names recur across
sections so the document reads as one world.

When the question under review is a *behavior* — a drag, a sheet, a scroll
conflict — consider building a small working simulator inside a phone frame
instead of static drawings. Letting the user feel the interaction settles
arguments that four static frames cannot.

## Feedback mechanics — the rules that protect the user's notes

The script stores stances and notes in localStorage keyed by
(`STORE_KEY`, each block's `data-item`). Consequences:

- **Within a round, never change a `data-item` key or the `STORE_KEY` after
  first publish.** Renaming a key orphans whatever the user already typed there.
  If a section's content evolves, update the visible text and leave the key
  alone, even if it becomes slightly stale as a label.
- Republish by writing the **same file path** and calling Artifact again — same
  URL, notes intact. Keep the favicon stable across republishes.
- Declare `capabilities: {downloads: true}` on first publish (omit on
  republish to carry it forward). The exporter falls back to clipboard copy
  when downloads are unavailable — the template handles both.

## Growing a round across sittings

Reviews grow — the user asks about one more surface and you add a section. Every
time you add to a published round, run a **consistency pass over the whole
document** before republishing:

- A new section may supersede part of an earlier one. Never leave both standing
  as if independent: mark the earlier claim *superseded, see §X* in place, and
  have the new section acknowledge what it changes. The document must read as
  one position, not a stack of sittings.
- Re-verify tense against the codebase — "Phase N will…" rots fast.
- Re-check cross-references, day-of-week on any date shown, and mock-data
  arithmetic in every phone, not just the new ones.
- Keep a running count in the lede honest, and note that sections are numbered
  in discussion order if a recommended build order lives elsewhere on the page.

## Round 2 and beyond

When the user says "done", read their exported notes file (or pasted notes).
Then build the next round as a new page state:

- Open with a **"Decided" ledger** — every item they stanced "Yes" moves into a
  ✓ grid of one-line commitments. Banked, out of discussion.
- Each "Discuss more" item gets a full section that quotes their questions from
  the notes and answers them one by one in the `qa` list, with reworked
  wireframes. Title it after their concern, not yours.
- "Not now" items get one line in the ledger area ("held; revisit when…") — do
  not relitigate them.
- A new round may use fresh `data-item` keys (prefix them, e.g. `R2-…`) since
  the prior round's notes were already exported.

## After the review: write the contract into the repo

A concluded review is not done when the last stance lands — **the decided
design must be distilled into the project's own docs at execution fidelity**,
because build sessions read the repo, not the artifact, and one-line roadmap
summaries force the next session to reinvent details as options (which is how
decided things get relitigated). For each reviewed item write, wherever the
project keeps phase-level detail: the decided rules and behaviors, the exact
copy strings, **the rejected options by name** (a rejection not written down
will be re-proposed as a fresh idea), and the genuinely-open items a build
session is allowed to ask about. Frame it as a contract: implementation
planning decides HOW, never reopens WHAT; new gaps get asked with an explicit
"not covered by the review" label. Wireframes described in precise words
outlast wireframes as pixels — an agent executes text better than it reads a
drawing, and earlier rounds' images may sit behind version history where a
fetch can't reach them.

## After the build: check the built screen against the review

A wireframe can only be wrong in the ways you thought to draw. Once the
reviewed work ships to a simulator or device, **run one verification pass
against the running app** — not a new review, a check that the contract was
kept. Do it while the build session's context is still warm, because the
cheapest time to fix a broken promise is before the next phase builds on it.

- **Read the screen as text, not as a picture.** An accessibility tree or
  element dump is cheaper than a screenshot and answers questions a screenshot
  cannot: whether a control has a label, whether a string is truncated rather
  than merely small, what the actual hierarchy is. Reserve screenshots for the
  things only pixels settle — spacing, contrast, whether it *looks* right.
- **Check the two failure classes wireframes structurally miss.** Accessibility
  (unlabeled controls, hit targets, focus order) and text that only breaks at
  the extremes — longest real string, largest Dynamic Type / system font scale,
  smallest supported screen. A mock is drawn at one size with the copy you
  chose, so it cannot show either. On iOS, `performAccessibilityAudit()` inside
  an existing UI test catches most of the first class for one line.
- **Diff against the decided rules, item by item.** Walk the contract written
  into the repo and mark each rule kept, missed, or changed-in-the-build. A
  rule the build quietly dropped is the same failure as a rejection nobody
  wrote down.
- **Report misses as facts, not as new proposals.** "E1 said one step per
  screen with the step's ingredients repeated; the build repeats all
  ingredients" — then let the user decide whether the build or the contract is
  wrong. Silently redesigning at this stage reopens a decided WHAT.

If the misses are substantial enough to need stances, they become a short
review round of their own: same page, same feedback mechanics, sections titled
after what broke.

## Publishing checklist

1. Validate the HTML before every publish — balanced tags at minimum (a quick
   parser script beats eyeballing 1,000 lines).
2. Publish via the Artifact tool from the scratchpad file; first publish sets
   `capabilities: {downloads: true}`, a stable favicon, and a version `label`.
3. In chat, lead with what changed and link the artifact — the page is the
   deliverable, the chat message is its cover letter. Restate the one or two
   questions you most want their eyes on.
