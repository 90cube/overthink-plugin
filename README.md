# Overthink

A Claude Code skill for **directed sequential diversity ideation**.

Instead of ten variations of the same idea, Overthink produces *N genuinely distinct ideas*
— each grown along its own problem-specific axis and forced away from every idea before it,
then tagged so you can pick one from a real spread.

## How it works

1. **Derive axes** — N dimensions along which solutions genuinely differ (mechanism, failure
   mode, cost structure, mental model…), tailored to your problem.
2. **Generate sequentially** — one idea per axis, each in a fresh pass that only knows it must
   differ from all prior ideas. This separation is the mechanism: it decorrelates the samples
   and prevents overlap.
3. **Classify** — every idea is tagged on two axes: quality (★ gem / ⚠ trap / · solid) and
   type (⚙️ mechanism / 🕹 interaction / 💡 concept), with a one-line reason.

The result is one fenced block per idea, with a vertical flow drawn for mechanism ideas,
ending in a recommended pick.

## Usage

After installing, invoke it with:

```
/overthink <your open-ended problem>
```

Or just say "overthink this" / "give me a few distinct ways to…".

**Skip it** for factual lookups, syntax questions, known-root-cause bugs, or closed phrasing
("quick", "standard", "just") — it costs ≈ N+2 model calls, so it earns its keep only when you
actually want a varied spread to choose from.

## Deep mode — `--deep`

```
/overthink --deep <one fixed goal>
```

Where plain Overthink fans out once over a single decision, `--deep` keeps going: it treats
the pick as settled, derives the *next* decision realizing the goal demands, fans out again,
and descends — concept → stack → architecture → interface → … — until nothing meaningful is
left to decide. Overthink living up to its name: it doesn't stop at one good answer, it thinks
the whole thing through to the bottom, by itself.

- **Autonomous by default** — no per-layer selection; the fixed goal is the objective function
  and the loop auto-picks (or fuses) the direction that best serves it. Add `--interactive` to
  hold the per-layer pick yourself.
- **Self-correcting** — when a deeper layer contradicts an upstream choice, it climbs back,
  re-decides, and re-descends (bounded, so it can't thrash).
- **Knows when to stop** — halts on a fixed point (the next layer adds no new constraint),
  diversity collapse, or a depth cap.
- **Auditable** — the whole decision tree, including the branches it *didn't* take, is written
  to `docs/overthink/<goal-slug>.md` so you can override a call or re-run a branch.

It is many times more expensive than a single Overthink run (each layer is itself a full
Overthink) — that depth is the point. Use it when you want one goal driven all the way to a
concrete, fully-decided plan.

## Full mode — `--full`

```
/overthink --full <one goal>
```

`--full` takes a goal not just to a set of *decisions* but all the way to the **finished
deliverable** — in whatever form that deliverable naturally takes. It hardcodes nothing about
the domain: a story, a song lyric, an ad's typography, a system design, a program — each grows
its own questions and its own output. A build spec is just the artifact you get *when the goal
is to build a program*, no more privileged than "the finished lyrics" for a song.

It wraps `--deep` in two domain-derived bookends:

1. **Framing gate (the one human touchpoint)** — up front, it surfaces the 3–5 load-bearing
   constraints the cascade can't safely guess (for a program: serverless vs self-hosted? for a
   lyric: genre, language, structure?), each with a recommended default so you can answer,
   override, or just say "use your defaults." It also settles what the finished artifact should
   be. This elicits constraints — it is *not* a per-step approval gate; once answered, the run
   is autonomous to the end.
2. **Artifact synthesis** — after the cascade converges, it derives the natural finished form
   for this goal and pours the locked decisions into it (the story's prose, the lyric's
   verse/chorus, the spec's sections), writing it beside the cascade state file.

The heaviest mode — a full `--deep` run plus framing plus synthesis. Use it when you want the
goal carried all the way to the actual finished thing, not just a plan.

## Installation

```bash
# 1. Register this repo as a marketplace
claude plugin marketplace add 90cube/overthink-plugin

# 2. Install the plugin from it
claude plugin install overthink@overthink-marketplace
```

## License

MIT
