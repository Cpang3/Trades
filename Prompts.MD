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

**Action:** Pull a fresh, no-cache snapshot of a named symbol — price, 24h volume, funding rate, open interest, key support/resistance, recent price chart, and any breaking news.
**Use when:** the user has a specific ticker in mind and wants the lay of the land before going deeper.
**Scope:** any tradeable symbol. Platform-aware — the user names which data platform to source from (Futu, Webull, YahooFinance, etc.).
**Variables:** `{TICKER}` (e.g. `AMD`, `BTC`), `{PLATFORM}` (e.g. `Futu`, `Webull`, `YahooFinance`).

### 2.1 — Fresh-fetch market snapshot

**Template (user pastes, substitutes `{TICKER}` and `{PLATFORM}`):**

> Market analysis of {TICKER} on {PLATFORM}. Do not reuse any previously cached prices or odds from this conversation. Fetch current price, 24h volume, funding rate, open interest, and key support/resistance levels. Chart the recent price action. Search for any breaking news or events affecting this asset right now.

**Expected agent output:**

1. **Chart** — 1-month price chart of the symbol (default horizon; user can request tighter/wider).
2. **Analysis** — short trend/context read.
3. **Market stats** — structured block:
   - Price
   - 24h volume
   - Open interest
   - Funding
   - Key support / resistance (tiered, from recent daily structure)
   - Trend / context (EMA structure, position in range)
4. **Breaking news / events** — surfaces today's catalyst if one exists; explicitly says "none" if not.
5. **Re-cut offer** — closes by offering tighter/different horizons.

**Reference output (example — $AMD):**

> Generates a graph of $AMD on a 1-month scale.
>
> **Market stats**
> - Price: $503.87
> - 24h volume: $21.30M
> - Open interest: $23.98M
> - Funding: +5.47% APR (≈ +0.0006%/hr, 1h cadence)
>
> **Key support / resistance (from recent daily structure)**
> - Support: $486–490, then $503–505, then $515–517
> - Resistance: $523–525, then $531–532, then $540+ (near recent highs)
>
> **Trend / context**
> - Short-term bias is slightly down (20EMA below 50EMA), but price is still near the upper end of the 1-month range.
>
> **Breaking news / events**
> - No clear, single "breaking" AMD-specific catalyst surfaced in the latest scan; most coverage is around the broader AI/datacenter GPU narrative rather than a discrete event today.
>
> The 1-month price chart is above. If you want tighter levels (e.g., last 10 trading days only) or a different horizon, I'll re-cut the levels.

---

## Prompt 3 — Deep Research

**Action:** Run a deep-web research sweep on a named symbol — breaking news, upcoming catalysts, whale / institutional activity, protocol changes, regulatory developments, and any off-chart signal that could move the price.
**Use when:** the user wants to validate or stress-test a thesis with off-chart information; specifically focused on what could move the price in the next 24–48 hours.
**Scope:** any tradeable symbol. Platform-aware — the user names which data platform to source from (Futu, Webull, YahooFinance, etc.).
**Variables:** `{TICKER}` (e.g. `AMD`, `BTC`), `{PLATFORM}` (e.g. `Futu`, `Webull`, `YahooFinance`).

### 3.1 — Off-chart catalyst sweep

**Template (user pastes, substitutes `{TICKER}` and `{PLATFORM}`):**

> Deep research {TICKER} on {PLATFORM}. Do not reuse any previously cached prices or odds from this conversation. Search the web for: breaking news, upcoming catalysts, whale activity, protocol changes, regulatory developments, and anything a trader needs to know that isn't visible on the chart. Focus on information that could move the price in the next 24–48 hours.

**Expected agent output:**

1. **Breaking news** — items published in the last 24–48 hours, each with named source + timestamp.
2. **Upcoming catalysts** — scheduled events in the next 7 days (earnings, FOMC, CPI, product launches, contract expirations, governance votes, etc.) with date/time.
3. **Whale / institutional activity** — large blocks, dark-pool prints, 13F changes, unusual options activity, on-chain transfers (for crypto).
4. **Protocol / corporate changes** — tokenomics changes, governance proposals, buybacks, splits, dilutions, M&A chatter.
5. **Regulatory developments** — agency actions, lawsuits, hearings, jurisdictional rule changes.
6. **24–48h price-moving read** — the agent's synthesis: which of the above is most likely to actually move the tape, ranked.
7. **Source list** — every claim links to a source; nothing assumed from memory.

**Reference output:** _to be captured from first canonical run._

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
