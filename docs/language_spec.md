# LoopScript Language Specification

**Version:** 0.1.0-draft  
**Status:** Work in progress — iterations will follow.

---

## Overview

LoopScript is a domain-specific language for defining, validating, and scoring cross-market trade loops. A `.loop` file describes a single arbitrage path. The engine validates it deterministically — it either passes or it doesn't.

---

## File extension

`.loop`

---

## Basic structure

Every loop file contains exactly one `loop` block.

```loop
loop <Name> {
  origin:     <COUNTRY_CODE>
  destination: <COUNTRY_CODE>
  commodity:  <category.subcategory.type>

  buy_price:  <amount> <CURRENCY>/<unit>
  sell_price: <amount> <CURRENCY>/<unit>

  costs {
    freight:  <amount> <CURRENCY>/<unit>
    tariff:   <ORIGIN>-><DESTINATION> @ <rate>%
    handling: <amount> <CURRENCY>/<unit>
  }

  constraints {
    min_margin:       <percent>%
    max_transit_days: <integer>
    min_volume:       <integer> <unit>
  }

  validate: margin > min_margin
  signal:   EXECUTE | WATCH | HOLD
}
```

---

## Types

| Type | Example | Notes |
|------|---------|-------|
| Country code | `TR`, `PL`, `UAE` | ISO 3166-1 alpha-2 or alpha-3 |
| Currency | `USD`, `EUR`, `AED` | ISO 4217 |
| Unit | `kg`, `mt`, `unit` | mt = metric ton |
| Commodity | `textiles.surplus.apparel` | dot-separated taxonomy |
| Rate | `12%` | percentage, no decimals yet |
| Signal | `EXECUTE`, `WATCH`, `HOLD` | output state of a validated loop |

---

## Commodity taxonomy (initial)

```
textiles
  └── surplus
        └── apparel
              └── footwear

commodities
  └── agricultural
        └── soy
              └── coffee
  └── metals
        └── precious
              └── gold
              └── silver

electronics
  └── consumer
        └── b-grade
              └── overrun

industrials
  └── machinery
  └── components
```

---

## Cost resolution

Costs are deducted from the spread in order:

1. `freight` — flat rate per unit
2. `tariff` — percentage applied to `buy_price`
3. `handling` — flat rate per unit

**Margin formula:**

```
gross_spread = sell_price - buy_price
total_costs  = freight + (buy_price * tariff_rate) + handling
net_margin   = (gross_spread - total_costs) / sell_price * 100
```

---

## Signals

| Signal | Meaning |
|--------|---------|
| `EXECUTE` | Loop is valid, margin clears all constraints — act |
| `WATCH` | Loop is valid but margin is within 2% of minimum — monitor |
| `HOLD` | Loop fails one or more constraints — do not act |

The engine assigns `HOLD` automatically on validation failure. `EXECUTE` and `WATCH` are only reachable on a passing loop.

---

## Constraints

| Key | Type | Description |
|-----|------|-------------|
| `min_margin` | percent | Minimum net margin required |
| `max_transit_days` | integer | Maximum acceptable transit time |
| `min_volume` | integer + unit | Minimum viable trade volume |

More constraint types coming in future versions.

---

## Validation rules

- `buy_price` must be less than `sell_price` before costs
- All `costs` fields are required in v0.1
- `origin` and `destination` must be different
- `commodity` must resolve to a known taxonomy node
- `validate` expression is evaluated last — must return true for EXECUTE or WATCH

---

## CLI reference

```bash
loopscript validate <file.loop>         # Validate a single loop
loopscript score <directory/>           # Score all loops in a folder
loopscript rank --min-margin <n>        # Rank by margin, filter by constraint
loopscript score <directory/> --output json
```

---

## Example

```loop
loop TurkeyTextileArb {
  origin:      TR
  destination: PL
  commodity:   textiles.surplus.apparel

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

---

## What's not in v0.1

- FX conversion (multi-currency cost resolution)
- Forward vs. spot price handling
- Multi-leg loops (A → B → C)
- Time-windowed constraints
- Live data binding

All of the above are planned. This spec covers the minimum viable language needed to parse, validate, and score a single-leg loop.

---

*LoopScript is the public research layer of LoopTrade. The execution engine is proprietary. The language is open.*
