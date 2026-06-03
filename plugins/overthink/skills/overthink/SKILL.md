---
name: overthink
description: Directed sequential diversity ideation. Derives N problem-specific axes, gets consent, then generates one idea per axis sequentially — each avoiding all prior ideas — and tags each trap/gem/solid. Use on /overthink, "overthink this", or open-ended "give me a few distinct ways to…" where the goal is to pick one idea from a varied spread. For "/overthink --deep", "deep overthink", "끝까지 과생각", or any request to take ONE fixed goal and autonomously work out every concrete decision needed to realize it (concept → stack → architecture → design → …), run the recursive autonomous concretization cascade in the Deep mode section instead of the single-pass flow. Skip for factual lookups, syntax, known-root-cause bugs, or closed phrasing ("quick", "standard", "just").
license: MIT
---

# Overthink

Generate a *spread* of genuinely distinct ideas so the user can pick one. Not ten
variations of the same idea — N ideas that each occupy a different region of the solution
space, because each is grown along its own axis and forced away from all the others.

## Pre-flight (always show first)

This is more expensive than a direct answer: about N+2 model calls. Before doing the work,
show the user this warning and the proposed axes, and wait for consent — UNLESS the user
explicitly typed `/overthink`, said "overthink this", or supplied their own axes.

> ⚠️ Overthinking runs N+2 reasoning passes (1 to design axes + N to generate + 1 to
> classify). Cost ≈ N× a single answer. I'll grow the idea set step by step.

If the request is a factual lookup, a syntax question, a bug with a known cause, or used
words like "quick / standard / just", do NOT use this skill — answer directly.

## Working language: caveman-English drafts, native final

Detect the user's request language at the start (the language of their problem). Then:

- **All intermediate work is in compressed English** — axis names + rationales, every
  sub-agent prompt and its returned idea, the EXCLUDE list, and the classification. Use
  terse, telegraphic English: fragments over sentences, drop filler/hedging, abbreviations
  fine. Preserve code, paths, and URLs verbatim. This is the cheap working medium and is
  never shown to the user.
- **Only the FINAL output is written in the user's detected language**, expanded back into
  clean, readable prose. You (the orchestrator) already write this in your own answer turn,
  so the translation/expansion costs **no extra model call** — it is free.
- Do **not** over-compress: drafts must stay precise enough that the exclude-comparison and
  the final synthesis never lose an idea's substance. Compress expression, never the idea.

(Concept borrowed from CAVEMAN: compress output *expression*, not *reasoning*.)

## The flow

### Phase 0 — Axes (plan)

Derive **N axes** (default 5) tailored to THIS problem and to what you know about the user
(their stated context / CLAUDE.md / intent). An axis is a *dimension along which solutions
differ* — mechanism, failure mode, cost structure, user served, assumption removed,
timescale — NOT surface wording. The axes must be maximally distinct from each other.
Name and describe the axes in compressed English (this is internal working text).

Present them and ask for consent:

> I'll explore along these 5 axes:
> 1. **{axis}** — {why it fits}
> 2. …
> Proceed, or want to swap any?

If the user supplied axes, skip derivation and use theirs. If they typed `/overthink`
explicitly, you may state the axes and proceed without a separate confirmation round.

### Phase 1 — Generate (run): one separate call per axis, in order

Generate the ideas as **N separate, sequential generation steps — NEVER all at once in a
single reply.** This separation IS the mechanism. If you write all N ideas yourself in one
pass, the tool collapses into plain single-turn brainstorming: the exclusion is no longer
enforced and each idea anchors on the ones you just wrote. Do not do that.

Loop over the axes in order (axis 1, then 2, … then N). For each axis:

1. **Build the EXCLUDE list** = a one-line summary of EVERY idea produced in the earlier
   steps. For axis 1 it is empty (`(none yet)`).

2. **Dispatch exactly ONE generation via the Agent/Task tool** — a fresh sample, so it does
   not inherit your own running reasoning. The sub-agent's prompt contains ONLY: the
   problem, the current axis + why it fits, and the EXCLUDE list. Instruction:

   > You are a generator, not a critic. Produce ONE idea for the problem, viewed through
   > this AXIS. Reply in compressed English — terse fragments, no filler; keep any code,
   > paths, or URLs verbatim. One phrase or sentence. It MUST be structurally different from
   > every idea in the EXCLUDE list — a different underlying approach, not just different
   > wording. Do not evaluate. Output the idea and a one-clause rationale.

3. **Append the returned idea to your running list BEFORE starting the next axis.**

Why a separate call each time (and why "inline, all in one pass" is wrong): a fresh
generation that only knows "produce something unlike these" samples a genuinely different
region of the space. The **fresh context decorrelates** the sample; the **EXCLUDE list
(passed in) prevents overlap**. Both are required — and neither happens if you produce the
whole set in one reply. Yes, this is N separate tool calls; that cost is expected and was
declared in the pre-flight warning. Do not "save cost" by collapsing the loop.

### Phase 2 — Classify (light convergence)

Once all N ideas exist, do ONE critic pass (in compressed English — internal working text).
Tag each idea on TWO independent axes:

**Quality** (how good):
- **★ gem** — non-obvious but genuinely viable (the overlooked one),
- **⚠ trap** — looks attractive but flawed (hidden cost, won't scale),
- **· solid** — sound, workable, conventional.

**Type** (what kind):
- **⚙️ mechanism** — functional, requires building several steps/stages to work,
- **🕹 interaction** — a UX gesture / surface; mostly presentational,
- **💡 concept** — a reframe / metaphor / mental-model shift.

Give each a one-line reason. No 0–10 scores — they are model-subjective and not trustworthy;
the class + reason is the honest signal.

**For every ⚙️ mechanism idea**, also extract its core steps as a short ordered list
(compressed English working text). These become a vertical arrow flow (`↓`) inside that
idea's code block in the final output — because a multi-stage functional idea is far easier
to grasp drawn than described. Do NOT draw flows for 🕹 interaction or 💡 concept ideas (a
flow there is noise); only mechanism ideas whose multiple steps genuinely need to be seen.

## Output shape

This is the ONLY user-facing part — write it in the user's detected language, expanded into
clean readable prose (the working drafts above were compressed English; expand here).

**One idea = one fenced code block.** Each idea gets its own ``` block — this keeps ideas
visually separated and guarantees all of an idea's lines (header, rationale, flow) stay in a
single rendering context (no rows splitting across markdown blocks). Every block opens with
both tags — quality (★/⚠/·) and type (⚙️/🕹/💡) — then the axis, idea, rationale, the
gem/trap "why", and — for ⚙️ mechanism ideas only — a vertical arrow flow.

First, one plain line: `Problem: <problem>`. Then one code block per idea:

Mechanism idea (with flow):
```
★ gem · ⚙️ mechanism · [axis-name]
<idea title>

<rationale, 1–2 lines>
→ why gem: <...>

흐름:                      (label in the user's language)
  <step 1>
    ↓
  <step 2>
    ↓
  <step 3>
```

Non-mechanism idea (no flow):
```
· solid · 🕹 interaction · [axis-name]
<idea title>

<rationale, 1 line>
```

Flow rules (⚙️ mechanism ideas only):
- **Vertical** chain with `↓` between steps — NO horizontal boxes (CJK label widths don't
  align in monospace and break the drawing).
- Branches: indent `├─` / `└─` under the step that forks.
- The flow lives INSIDE that idea's own code block — never split it out.
- Keep step labels short; keep code/paths/URLs verbatim.
- No flow for 🕹 interaction or 💡 concept ideas.

After all idea-blocks, one plain line (not in a block): which you'd pick to run with, and why.

## Calibration — length follows quality, not the reverse

caveman compression applies to **internal drafts only**; never carry that terseness into the
final, user-facing output. The final output is judged on quality and clarity, and the two
levers act on different stages — so compress the drafts AND invest in the final; it is not a
trade-off.

- Spend words where they earn their keep: a ⚙️ mechanism idea earns its flow + a fuller
  "how it works"; a 🕹 interaction or 💡 concept idea stays tight (a line or two).
- The ★ gem and the final pick deserve the most depth; ⚠ traps get a crisp one-line "why";
  · solids stay brief.
- Cut filler, hedging, and restatement — but never cut substance, a needed flow, or the
  reasoning that makes a judgment trustworthy.
- If trimming would make an idea harder to act on, keep it. In the final output,
  **quality > token count.** Slightly longer is fine when it buys clarity.

## Deep mode (`--deep`) — recursive autonomous concretization

Trigger this mode on `/overthink --deep <goal>`, "deep overthink", "끝까지 과생각", or any
request to take ONE fixed goal and autonomously work out *every* concrete decision needed to
realize it. Add `--interactive` (`/overthink --deep --interactive <goal>`) to keep the loop
but hand the per-layer pick back to the user instead of auto-selecting.

**Terms used below:** *goal* = the user's one fixed purpose, set once at invocation and never
changed (it plays the role that `Problem:` plays in the single-pass flow, but for the whole
run). *Layer* = one step of the cascade (concept, then stack, then …), each answering its own
derived question. *Axis* = a within-layer dimension of the single-pass flow (unchanged
meaning). Don't conflate a cascade *layer* with an idea's internal *stage*.

**Consent is given once, up front.** Invoking `--deep` IS the consent — so the per-layer runs
do NOT re-show the Pre-flight warning or run Phase 0's "proceed?" gate (that would stall the
cascade at every layer and kill the autonomy). Show the cost/depth-cap line once before the
first layer (see Cost), then run uninterrupted to a stop condition.

Plain overthink fans out once over a single decision and hands you a spread to pick from.
`--deep` does not stop there: it treats the spread-pick as settled, derives the *next*
decision that realizing the goal now demands, fans out again, and descends —
concept → stack → architecture → interface → … — until nothing meaningful is left to decide.
This is overthink living up to its name: it doesn't stop at one good answer, it thinks the
whole thing through to the bottom, by itself.

### The cascade

The fixed **goal** (the user's original purpose) is the north star for the entire run —
every choice is judged by "does this best serve that goal, given what's already decided." The
goal never changes; everything below it does.

Each layer of the cascade:

1. **Run the normal single-pass flow** (Phase 0–2) on the *current* concretization question —
   derive axes, generate N ideas sequentially with the EXCLUDE list, classify. Nothing about
   that flow changes; deep mode just invokes it repeatedly on a moving question.
2. **Auto-select — no human gate (this is the default).** Deep mode makes the pick itself:
   take the ★ gem, or *fuse* the 2–3 ideas that together best serve the goal into one coherent
   decision. The goal is the objective function. With `--interactive`, pause here and let the
   user pick or fuse instead, then continue automatically — the loop machinery is identical;
   only who holds the selector changes.
3. **Record** the decision (with its fit-to-goal reason) and the *unchosen* branches to the
   state file (below).
4. **Derive the next layer emergently.** Given the goal + the decision path so far, what is the
   single most important still-undecided question? Let it grow out of the last decision
   ("stack = React" → next is "structure that fits React"), biased toward — but never locked
   to — the natural build pipeline (concept → tech → structure → data → interface → polish →
   ship). A game, a CLI, and an essay will grow different chains; that divergence is the point.

### Self-correction — climb-back

A one-way greedy descent compounds error: a wrong pick at an upper layer silently poisons
everything beneath it, and with no human gate nothing catches it. So the descent is **not**
strictly one-way.

When a deeper layer surfaces a genuine *contradiction* with an upstream decision (e.g. "this
interaction is impossible on the stack we chose"), climb back to that upstream layer,
re-select with the new information, and re-descend from there. This is the Ralph principle —
failures are data, and the persisted state lets each pass read and revise its own prior work.
It is what makes "no human needed" produce something sound instead of a tall stack of
compounding mistakes.

Mechanics of a climb-back, so the loop stays well-defined:

- **Supersede, don't erase.** Append a new revised entry for the climbed-back layer that
  *supersedes* the stale one (mark the old one superseded); the append-only state file keeps
  both so the history stays auditable.
- **Re-descend regenerates below.** Every layer below the climb-back point was derived from the
  now-changed decision, so discard those decisions and regenerate them fresh from the revised
  layer down. Layers above are untouched.
- **Guard against thrash.** Climb back only on a real contradiction, never on mild preference.
  Cap it: if the *same* upstream layer is climbed back to more than twice, stop instead and
  report the unresolved tension — repeatedly returning to one layer means the goal itself is
  over-constrained, and that is a finding, not a failure to fix in-loop.

### When to stop — fixed-point + convergence

Halt as soon as ANY of these fires:

- **Fixed point (primary) — checked at the derivation step (step 4).** After recording a
  decision, try to derive the next still-undecided question. If twice in a row that derivation
  can only surface a question that adds **no new material constraint** on realizing the goal —
  i.e. the honest answer to "what's left to decide?" is "nothing that changes the build, only
  cosmetic refinement" — you are at a fixed point. Stop. (This is about the *next question
  failing to appear*, distinct from Diversity collapse below, which is about the *answers
  within a layer* failing to differ.)
- **Diversity collapse — checked at the generation step.** Inside a layer, the N generated
  ideas are no longer meaningfully distinct: the spread overthink depends on has died, so
  there is nothing left to explore at this granularity. Stop.
- **Max depth (backstop).** A hard cap (default 8 layers) so an over-eager cascade can't run
  forever. This is a safety net; the two convergence checks above should normally fire first.

### State file — the cascade lives on disk

Persist the run to `docs/overthink/<goal-slug>.md`, relative to the project root (the repo root
if in one, otherwise the current working directory); create the directory if absent. The file
IS the state: the run becomes resumable and auditable, and a climb-back can re-read what was
decided before. Append as you go — never rewrite history (climb-backs supersede, per above).

Record per layer: the layer's question, the N generated ideas with their tags, the decision
(pick or fusion) plus its fit-to-goal reason, and the **unchosen branches kept verbatim** —
they are the only record of what the autonomous selector passed over, so the user can later
override a call or re-run a branch.

### Output

When the cascade halts, present in the user's language:

1. **The decision spine**, top to bottom — goal → each layer's question → the decision + why.
   This is the headline result.
2. **Notable unchosen branches** per layer, as a secondary/collapsed list — what it considered
   and set aside.
3. A pointer to the state file, and **which stop condition fired** (fixed point / diversity
   collapse / depth cap).

### Cost

A deep run is many times more expensive than plain overthink — each layer is itself a full
N+2-pass overthink, multiplied by cascade depth, plus any climb-backs. That cost is the whole
point of `--deep`; do not silently downgrade to a single shallow pass to save calls. But
because it is expensive, state the depth cap up front, and do not start a deep run for a
request that a single overthink — or a direct answer — would already serve.

## Anti-patterns

- **Single-turn collapse (the main one).** Writing all N ideas in one reply instead of N
  separate, sequential Agent generations. It looks the same, but the exclusion is no longer
  enforced and the samples anchor on each other — this is exactly the cheaper method this
  tool is built to beat. Always dispatch one generation per axis, feeding the accumulated
  EXCLUDE list each time.
- Generating in parallel without the exclude list → ideas overlap; the spread collapses.
- Axes that differ only in wording → derive axes by *dimension*, not synonym.
- Turning the single-pass flow into full scoring/clustering/deepening → out of scope for plain
  overthink; if the user wants the goal driven all the way down, that is what `--deep` is for.
- In `--deep`, climbing back on mild preference instead of real contradiction → the cascade
  never settles. Climb back only when a downstream layer makes an upstream decision untenable.
- In `--deep`, silently collapsing the per-layer cascade into one shallow pass to save calls →
  defeats the whole mode. The depth is the point; declare the cap up front instead.
