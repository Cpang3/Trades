# Trades — the Brain

The trading policy for a retail, signal-driven, directional **options / equity swing-trading** system
(daily charts, US Tier-1 large-caps, executed paper-first on Futu).

This repo is **the brain** — the canonical source of truth for *what* to trade, *what rules* to
enforce, and *why*. These five `.MD` documents own the strategy. The **mechanism** that enforces them
(the code, the Futu broker rails, the policy-gate engine) lives in a separate repo —
[`Cpang3/tradingbot`](https://github.com/Cpang3/tradingbot) — governed by its `STRATEGY.md`
(architecture blueprint) and `CLAUDE.md` (engineering logbook). The mechanism *references* this brain;
it never restates it. `watchlist.yaml` in that repo is the machine-readable form of `watchlist.MD`
here.

**Strategy in one sentence:** Daily-chart swing trading on Tier-1 US large-caps, expressed as long
shares or long calls/puts, with entries gated by a multi-factor screener built on the **Enhanced
Mechanic System (EMS)** and a regime classifier built on **Markov / HMM** analysis of returns. The
bot is the **discipline layer** — it executes only when the rules allow.

## The five documents

| # | Document | Covers |
|---|---|---|
| 1 | [watchlist.MD](./watchlist.MD) | Three-tier universe (1A/1B/1C), override discipline, hard exclusions, and the full 68-name themed watchlist. |
| 2 | [Trading Rules.MD](./Trading%20Rules.MD) | Buying-vs-trading philosophy, HK execution windows, capital-efficiency window, anti-tilt cooldowns, the 14 red lines, the 8-gate pre-trade checklist, journaling. |
| 3 | [Macro Indicators.MD](./Macro%20Indicators.MD) | Bot 1 mode classifier (VIX / yields / CPI / PPI / FOMC → trading mode), the Markov/HMM regime model, and macro-event blocks. |
| 4 | [Technical Analysis.MD](./Technical%20Analysis.MD) | The EMS chart layer — A+ setups, support/resistance, flag/flagpole, volume confirmation, the Gate-5 verification checklist. |
| 5 | [Trading Strategy.MD](./Trading%20Strategy.MD) | Tier-scaled ±50% sizing, 50%-profit/50%-withdrawal, 100%→full-exit, the exit waterfall, shares-vs-options selection, and the Bot 2 screener scoring. |

## How they fit together (the decision flow)

```
Macro Indicators  →  Bot 1 sets the day's MODE (what shape of trade is allowed)
        │
        ▼
Watchlist         →  Is the name on the list? Which tier? (sizing + gate friction)
        │
        ▼
Technical Analysis→  Is there an A+ EMS setup on the daily chart? (Gate 5)
        │
        ▼
Trading Strategy  →  Bot 2 grades it, sizes it, picks shares vs options, sets exits
        │
        ▼
Trading Rules     →  8 gates enforced in order; cooldowns / red lines can SKIP at any step
```

## Honesty notes carried in the docs

- **Bot 1 (regime → mode) is designed and coded but not yet integrated** into the live gate flow;
  the day's mode is supplied manually until it's wired in.
- Every watchlist thesis tagged **`[DRAFT — review]`** was drafted from public info and needs
  user sign-off before live trading.
- **No real-money trades for the first 4 weeks** — paper only, by rule.

---

*Attribution: the EMS pattern framework is adapted from EnhancedMarket / TradeProElite, "Day Trading
101 — From Beginner to Expert," for daily-chart swing trading. Stock selection is the user's own
research edge; the discipline overlays come from the user's diagnosed failure modes.*
