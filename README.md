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

## Installation

```bash
# 1. Register this repo as a marketplace
claude plugin marketplace add 90cube/overthink-plugin

# 2. Install the plugin from it
claude plugin install overthink@overthink-marketplace
```

## License

MIT
