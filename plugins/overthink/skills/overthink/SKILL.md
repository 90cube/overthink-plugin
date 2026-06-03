---
name: overthink
description: Directed sequential diversity ideation. Derives N problem-specific axes, gets consent, then generates one idea per axis sequentially — each avoiding all prior ideas — and tags each trap/gem/solid. Use on /overthink, "overthink this", or open-ended "give me a few distinct ways to…" where the goal is to pick one idea from a varied spread. Skip for factual lookups, syntax, known-root-cause bugs, or closed phrasing ("quick", "standard", "just").
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

## Anti-patterns

- **Single-turn collapse (the main one).** Writing all N ideas in one reply instead of N
  separate, sequential Agent generations. It looks the same, but the exclusion is no longer
  enforced and the samples anchor on each other — this is exactly the cheaper method this
  tool is built to beat. Always dispatch one generation per axis, feeding the accumulated
  EXCLUDE list each time.
- Generating in parallel without the exclude list → ideas overlap; the spread collapses.
- Axes that differ only in wording → derive axes by *dimension*, not synonym.
- Turning this into ADHD's full scoring/clustering/deepening → out of scope; this is the
  light version. Pick one idea and move.

## Companion code

A TypeScript library + CLI does the same loop with structured JSON and state files
(`overthink plan` → `overthink run`). Use it for batch/scripted runs outside Claude.
