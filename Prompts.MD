# Prompts

User-facing prompt templates for invoking actions from the trading agent.
Each numbered section = one type of action the user can trigger.
The workflow is not strictly linear — pick the prompt that matches the user's intent at that moment.

---

## Prompt 1 — Trade Ideas

**Action:** Surface candidate trades for the user.

### 1.1 — Watchlist-scoped one-cycle swing ideas (Stocks)

**Use when:** the user is out of original trade ideas and wants a templified list of candidates drawn from the existing watchlist.
**Scope:** stocks listed in `watchlist.MD` (this repo) / `watchlist.yaml` (tradingbot repo) only — the agent should NOT pull from the broader market.
**Horizon:** one standard monthly options cycle (~19–22 trading days, ~21 on average).
**Output convention:** each idea should include thesis, trigger, invalidation, and target sized for the one-cycle hold.

Templates the user can paste/select:

1. "Give me 3 swing-long ideas in U.S. stocks designed for a one-cycle hold (about 19–22 trading days) with clear invalidation."
2. "Scan for stocks breaking out of 4–8 week bases that fit a one-cycle swing."
3. "Find 5 swing candidates with improving relative strength vs the S&P 500 over the last month."
4. "Which large-caps have a fresh higher-low structure and room to 2R within one monthly cycle?"
5. "Give me 3 swing-short ideas where the daily trend just rolled over for a one-cycle hold."
6. "Find pullback-to-trend setups: price above 50-DMA, pulling back toward support, with a 19–22 day hold."
7. "Identify 3 earnings-driven swing trades for the next monthly cycle with a clear post-event plan."
8. "Show me 5 stocks with tight consolidation and rising volume — likely 1-cycle breakouts."
9. "Give me swing ideas where price reclaims a key level (prior high / 50-DMA / 200-DMA) and has room for 19–22 days."
10. "Find 3 stocks with bullish weekly structure and a near-term trigger suited to a one-cycle hold."
11. "Suggest 2 sector-rotation swing trades based on relative strength this month."
12. "Which stocks show a clean trend + momentum reset (RSI cooled, structure intact) for a one-cycle swing?"
13. "Find 3 swing setups with 2.5R+ potential and tight stops within ~21 trading days."
14. "Give me 5 swing candidates in high-liquidity names (>$2B avg daily volume) for a one-cycle hold."
15. "Identify 3 'failed breakdown → reversal' swing setups on the daily chart for a 19–22 day window."
16. "Find swing trades with a gap-fill or gap-hold thesis for the next monthly cycle."
17. "Give me 3 swing ideas in defensive names for a one-cycle hold."
18. "Scan for stocks with strong institutional accumulation signals (volume + price structure) suited to a 1-cycle swing."
19. "Find 3 mean-reversion swing longs: oversold bounce into major support over the next monthly cycle."
20. "Give me 2 swing trades tied to macro catalysts (rates, CPI, FOMC) with tight risk for a 19–22 day hold."
21. "Identify 3 bull-flag swing setups with a defined trigger and target for a one-cycle trade."
22. "Find 3 swing shorts in weak sectors where rallies are failing at resistance over a one-cycle window."
23. "Give me 5 charts with expanding volatility that favor 1-cycle swings."
24. "Find 3 stocks with fresh trend change (higher high + higher low) in the last 15 days, ready for a one-cycle hold."
25. "Give me a watchlist of 10 swing candidates with entry, stop, and one-cycle target (19–22 trading days)."

---

## Prompt 2 — Market Analysis

**Action:** Pull a snapshot view of a named symbol — price, volume, funding, key technical levels, recent news.
**Use when:** the user has a specific ticker in mind and wants the lay of the land before going deeper.

_Templates to be added._

---

## Prompt 3 — Deep Research

**Action:** Investigate a named symbol in depth — breaking news, catalysts, whale / institutional / unusual options activity.
**Use when:** the user wants to validate or stress-test a thesis with off-chart signals.

_Templates to be added._

---

## Prompt 4 — Trade Idea (Concrete Spec)

**Action:** Convert a thesis into a tradeable plan — entry, target, stop, position size, R/R.
**Use when:** the user has converged on a name and wants the executable version of the idea.

_Templates to be added._

---

## Prompt 5 — Backtest Strategy

**Action:** Run a simple strategy backtest and serve a dashboard with the results.
**Use when:** the user wants to pressure-test a setup pattern or rule across history before committing capital.

_Templates to be added._

---

## Prompt 6 — Quick Summary

**Action:** One-line / one-screen view — price, day-over-day change, recent news.
**Use when:** the user wants a fast pulse-check on a name without full analysis.

_Templates to be added._
