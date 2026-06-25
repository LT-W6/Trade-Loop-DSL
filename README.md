# LoopScript

**A domain-specific language for defining, validating, and scoring cross-market trade loops.**

Built in Rust. Developed by [LT-DC](https://github.com/LT-W6) — the research and infrastructure arm of LoopTrade.

---

## What is a trade loop?

A trade loop is a structured arbitrage path — a defined sequence of buy, move, and sell operations across geographies, commodities, or market segments where a pricing inefficiency creates a capturable spread.

Markets are fragmented. Information asymmetry between buyers and sellers creates persistent gaps across geographies, regulatory regimes, and logistics networks. These gaps are not random — they are structural. LoopScript gives you a formal language to describe them, validate them, and score them systematically.

---

## What is LoopScript?

LoopScript is a DSL (domain-specific language) for trade loop specification. A `.loop` file describes a single arbitrage path — its origin, destination, commodity, cost structure, margin constraints, and execution signals.

The Rust engine parses, validates, and scores `.loop` files deterministically. No spreadsheets. No ambiguity. A loop either passes or it doesn't — and when it passes, you know exactly why.

```loop
loop TurkeyTextileArb {
  origin: TR
  destination: PL
  commodity: textiles.surplus.apparel

  buy_price:  2.40 USD/kg
  sell_price: 4.10 USD/kg

  costs {
    freight:  0.30 USD/kg
    tariff:   TR->PL @ 12%
    handling: 0.08 USD/kg
  }

  constraints {
    min_margin:       18%
    max_transit_days: 21
    min_volume:       5000 kg
  }

  validate: margin > min_margin
  signal:   EXECUTE
}
```

```bash
$ loopscript validate turkey_textile.loop

  Loop:       TurkeyTextileArb
  Spread:     $1.70/kg
  Net margin: 23.4%
  Status:     ✓ VALID — signal: EXECUTE
```

---

## Core concepts

**Loop** — the top-level unit. One loop describes one arbitrage path.

**Costs** — structured deductions: freight, tariff, handling, FX conversion. Tariffs resolve against real trade routes using the `origin->destination @ rate` syntax.

**Constraints** — the conditions a loop must satisfy to signal. Minimum margin, maximum transit time, minimum volume. If any constraint fails, the loop is invalid.

**Signal** — the output state of a validated loop: `EXECUTE`, `WATCH`, or `HOLD`. Your decision logic, encoded.

---

## CLI

```bash
# Validate a single loop file
loopscript validate path/to/loop.loop

# Score all loops in a directory
loopscript score ./loops/

# Rank loops by net margin, filtered by constraints
loopscript rank --min-margin 15 --max-transit 30

# Output results as JSON
loopscript score ./loops/ --output json
```

---

## Installation

Requires Rust 1.75+.

```bash
git clone https://github.com/LT-DC/loopscript
cd loopscript
cargo build --release
cargo install --path .
```

---

## Project structure

```
loopscript/
├── src/
│   ├── lexer.rs       # Tokenizer
│   ├── parser.rs      # AST construction
│   ├── validator.rs   # Constraint resolution and margin calculation
│   ├── scorer.rs      # Loop ranking and signal generation
│   └── cli.rs         # Command-line interface
├── examples/
│   ├── turkey_textile.loop
│   ├── uae_gold_arb.loop
│   └── brazil_soy_rotation.loop
├── docs/
│   └── language_spec.md
└── tests/
```

---

## Example loops

The `/examples` directory contains real-world inspired loops across the sectors LoopTrade operates in. They are illustrative — commodity prices and tariff rates are representative, not live. Use them to understand the language, build your own, and contribute.

Current examples:
- **Turkey → Eastern Europe** — textile surplus arbitrage
- **UAE gold souk** — precious metals volatility windows
- **Brazil soy** — cooperative vs. futures pricing differential
- **Vietnam electronics** — factory overrun B-grade inventory

---

## Language specification

Full language spec lives in [`docs/language_spec.md`](docs/language_spec.md).

It covers: type system, commodity taxonomy, tariff resolution, FX handling, constraint grammar, signal states, and the validation pipeline.

---

## Contributing

LoopScript is the public research layer of LoopTrade's infrastructure. The execution engine is proprietary. This language, its parser, validator, and scoring logic are open for contribution.

Good places to start:

- Add new commodity categories to the taxonomy
- Improve tariff resolution logic
- Build a new constraint type
- Write example loops for markets not yet covered
- Improve error messages (Rust's parse errors should be friendly)

Open an issue before a large PR. We review everything.

---

## Why Rust?

Trade loop validation is deterministic computation. It should be fast, correct, and explicit about failure. Rust gives us all three — and the type system forces the kind of rigor that financial logic demands.

There is no runtime ambiguity in a `.loop` file. Either the math closes or it doesn't.

---

## About LT-DC

LT-DC is the research and infrastructure division of LoopTrade — a cross-market arbitrage platform that allocates capital dynamically across commodities, surplus inventory, logistics, and cross-border trade systems across 40+ countries.

LoopScript is how we think in public.

---

## License

MIT — see [LICENSE](LICENSE).

---

*Built with precision. Loops don't lie.*
