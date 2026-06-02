# Technical Analysis (EMS)

> **Compartmentalized from `STRATEGY.md` v0.3 §3 (the EMS backbone) + Gate 5 of the swing-skill
> pre-trade checklist.**
> This file is the **chart layer**: the Enhanced Mechanic System (EMS) pattern rules that decide
> whether a setup is A+ and tradeable. Price action and volume **only**.
>
> **Attribution:** the EMS rules are reproduced from EnhancedMarket (TradeProElite), "Day Trading
> 101 — From Beginner to Expert", and **adapted for daily-chart swing trading** rather than intraday.
> Modifications from the original are flagged **[ADAPTED]**.
>
> Part of a 5-file set: `watchlist.MD` · `Trading Rules.MD` · `Macro Indicators.MD` · `Technical Analysis.MD` · `Trading Strategy.MD`

---

## 1. Trade A+ setups only (STRATEGY.md §3.1)

Only trade setups meeting strict A+ criteria. An A+ setup requires **all** of:

- A clean **support / resistance** level — multiple touches, established structure
- A valid **flag / wedge** pattern with a flagpole — sharp directional move + tight consolidation
- **Volume confirms** — dry-up in the consolidation, expansion on the breakout

If the setup is not A+, **no trade.** Quality over quantity. This is the single most important
technical rule.

---

## 2. Trade based on charts, not P&L (STRATEGY.md §3.2)

Decisions come from chart structure, not account balance or recent results. P&L-driven decisions are
emotional decisions. The chart either presents an A+ setup or it doesn't — that's the only input to
entry.

---

## 3. Stops are chart-based, not percentage-based (STRATEGY.md §3.3)

Stops sit at meaningful chart levels:

- **Long entries:** stop below key support **OR** below the flag/wedge low
- **Short entries:** stop above resistance **OR** above the flag/wedge high

**Do NOT use fixed-% stops** (e.g., "-5%"). Use chart structure — the chart tells you where the
thesis is broken.

> Tier 1C exception (overlay, not EMS): Tier 1C trades use a chart-based stop **AND** a −25% premium
> stop, whichever fires first. See `Trading Strategy.MD`.

---

## 4. Daily charts are the decision timeframe **[ADAPTED]** (STRATEGY.md §3.4)

**Original EMS:** 30-min and 1-hour charts, specific intraday US session windows.

**Adapted:** **Daily charts are the decision timeframe.** Pattern definitions (S/R, flag, wedge,
volume) translate directly. All ENTER / SKIP / target / stop decisions reference the **daily bar** —
never 5/15/30-min.

The user's active window (HK 21:30–01:00) is for **executing and managing** decisions already made
on the daily — not for switching to intraday charts to find opportunities the daily didn't see.

**EMS's morning-session emphasis IS honored:** the preferred entry sub-window (HK 21:30–23:00) maps
to EMS's high-edge 09:30–11:00 EDT window. The afternoon EMS window (14:45–16:00 EDT) is dropped —
the user is asleep then. Pre-market and management-window **exits** are allowed; pre-market
**entries** remain forbidden (execution quality is poor).

---

## 5. Flag patterns with flagpole (STRATEGY.md §3.7)

The bread-and-butter setup. Definition:

- **Flagpole:** sharp directional move, minimum **+10% in 5–10 daily sessions** (long flagpole =
  bullish flag; short flagpole = bearish flag)
- **Flag body:** consolidation, range **≤ 50% of flagpole height**, duration **3–15 sessions**
- **Volume profile:** **contraction during the flag body, expansion on the breakout**
- **Entry:** breakout above the flag high (long) or below the flag low (short), **with volume
  confirmation**

**A flag without volume confirmation is not an A+ setup.**

---

## 6. No indicators except volume (STRATEGY.md §3.8)

Price action and volume only.

**Explicitly excluded:**
- Moving averages as primary signals (soft reference only; not a screener factor)
- VWAP
- RSI, MACD, Stochastic, Bollinger Bands
- Ichimoku, Fibonacci
- Level 2 / order book
- Sentiment / chatter scoring beyond the user's KOL judgment

**Explicitly included:**
- **Volume** — mandatory, in two places: pattern formation (dry-up) + breakout confirmation
  (expansion)
- **IV rank** — for options sizing and the shares-vs-options decision (a pricing input, not a
  technical indicator)

---

## 7. Gate 5 — A+ setup verification checklist

This is the operational form of the EMS rules, run as Gate 5 of the pre-trade checklist. The trade
must clear **all** of these on the **daily** chart:

1. **Support / resistance interaction OR flag/wedge with flagpole** present?
2. **If flag/wedge:** flagpole ≥ 10% in 5–10 sessions, flag body range ≤ 50% of flagpole height,
   3–15 sessions in formation?
3. **Volume profile:** contraction during the base/consolidation AND expansion on the breakout?
4. **Chart-based stop level identified** (below key support / flag low — NOT a fixed % stop)?

If any answer is "no" or "unclear" → **WAIT.** Do not enter on a "kinda" setup. EMS Rule 1: A+ only.

The technical structure carries **60% of the Bot 2 screener weight** (Relative Strength 20% + Base
Structure/Tightness 20% + Pivot/Pattern Behavior 20%), with Volume Behavior another 10%. See
`Trading Strategy.MD` for the full scoring table. Failure flags relevant to this gate:

| Flag | Grade cap |
|---|---|
| No A+ setup detected (no S/R interaction AND no flag/wedge) | D max |
| Volume not confirming | C max |
| RS rating < 30 | C max |

---

## 8. `patterns_to_watch` vocabulary

Each watchlist name carries a `patterns_to_watch` list. The vocabulary in use:

| Pattern | Meaning |
|---|---|
| `flag_breakout` | Breakout above a flag high (long) after a valid flagpole + dry-up consolidation |
| `support_hold` | Price tests and holds an established support level (multiple touches) |
| `pivot_reclaim` | Price reclaims a prior pivot/breakout level after a pullback (Minervini-style pivot) |

These tag *what to look for* on each name; they do not pre-authorize a trade — the daily chart still
has to present the A+ setup at decision time.

---

## 9. Chart-reading capability (operational note)

The advisor **can read chart screenshots** — candlestick bodies/wicks (including doji),
flag/wedge/flagpole geometry, support/resistance lines, and relative volume bars — when the user
pastes a daily-chart image. This is how Gate 5 is evaluated when the user brings a chart rather than
a structured data feed. It is **visual interpretation of a pasted image**, not a live OCR feed off a
broker terminal; the daily bar remains the unit of decision.

---

## 10. EMS discipline reminders (STRATEGY.md §3.10)

- Do not adjust stops mid-trade unless the **chart structure** changes (e.g., a new higher low forms
  in a long position).
- Limit number of trades — **A+ only**, quality over quantity.
- The chart is the only input to entry. If it's not A+, it's not a trade.
