# Prompts

User-facing prompt templates for invoking actions from the trading agent.
Each numbered section = one type of action the user can trigger.
The workflow is not strictly linear — pick the prompt that matches the user's intent at that moment.

---

## 🌐 Source Policy (global — applies to every prompt that touches news, sentiment, or catalysts)

**Preferred sources, in priority order:**

1. **Primary sources** — company IR pages, SEC filings (8-K, 10-Q, S-1), official press releases, conference-call transcripts.
2. **Reddit** — ticker-specific subreddits first, then general retail subs:
   - Ticker-specific: `r/{TICKER}` if it exists (`r/NVDA`, `r/AMD_Stock`, `r/TSLA`, `r/GME`, etc.)
   - General: `r/wallstreetbets`, `r/stocks`, `r/options`, `r/investing`, `r/SecurityAnalysis`
3. **News wires** — Reuters, Bloomberg, AP — for confirmed events only (use sparingly).
4. **Social (named accounts only)** — Twitter/X handles the user has flagged as KOLs. No anonymous bot-amplified posts.

**Excluded sources — do not fetch, do not cite:**

- Seeking Alpha
- Motley Fool
- Zacks
- Investor's Business Daily op-eds
- Yahoo Finance / MarketWatch editorial content (raw price/quote data is fine; editorial articles are not)
- AI-generated content farms and SEO aggregators
- Any source whose business model is rage-driven engagement

**Rationale:** retail-driven subs surface flow and sentiment before aggregators repackage it. Primary sources are the ground truth. Aggregators add lag, recycle stale narratives, and pollute the catalyst signal. The agent wants signal, not SEO.

---

## Prompt 1 — Trade Ideas

**Action:** Surface candidate trades for the user.

### 1.1 — Watchlist-scoped one-cycle swing ideas (Stocks)

**Use when:** the user is out of original trade ideas and wants a templified list of candidates drawn from the existing watchlist.
**Scope:** stocks listed in `watchlist.MD` (this repo) / `watchlist.yaml` (tradingbot repo) only — the agent should NOT pull from the broader market.
**Horizon:** one standard monthly options cycle (~19–22 trading days, ~21 on average).

**Default output shape — 7 stocks, tier-diversified, ranked:**

- **3 from Tier 1A** (mega-cap quality — NVDA, AMD, GOOGL, META, AAPL, MSFT, AMZN, TSLA, SPY, QQQ)
- **3 from Tier 1B** (narrative growth, auto-pass numeric filters)
- **1 from Tier 1C** (override candidate — strongest of the moment, fully respecting `watchlist.MD` §2 override discipline)

**Ranking:** the 7 ideas are ordered top → bottom by overall **preference / advantage**, computed as:

1. **Setup quality** — grade from `Trading Strategy.MD §7` screener (A > B; no C / D enter the list)
2. **Style multiplier** — ×1.10 momentum breakout / ×1.00 trend-following / ×0.90 mean-reversion (from §7)
3. **R:R for the one-cycle hold** — favor ≥ 2.0R
4. **Conviction qualifiers** — clean invalidation, no imminent earnings/macro within the cycle window, liquidity headroom

Rank 1 is the most actionable trade right now, regardless of tier. Rank 7 is still tradeable but lower priority. Each idea includes thesis, setup style, trigger, invalidation, and one-cycle target.

**Override:** if the user explicitly specifies a different count or tier mix in their prompt (e.g., "give me 3 ideas from Tier 1A only", "show 10 shorts", "skip Tier 1C this week"), the override wins. With no override, return 7 stocks 3/3/1 ranked.

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

> Market analysis of {TICKER} on {PLATFORM}. Do not reuse any previously cached prices or odds from this conversation. Fetch current price, 24h volume, funding rate, open interest, and key support/resistance levels. Chart the recent price action. Search for any breaking news or events affecting this asset right now — **sources per Source Policy: Reddit-first (`r/{TICKER}` + general subs), then primary sources; no aggregators (Seeking Alpha / Motley Fool / Zacks).**

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

> Deep research {TICKER} on {PLATFORM}. Do not reuse any previously cached prices or odds from this conversation. Search the web for: breaking news, upcoming catalysts, whale activity, protocol changes, regulatory developments, and anything a trader needs to know that isn't visible on the chart. Focus on information that could move the price in the next 24–48 hours. **Sources per Source Policy: primary (IR / SEC) + Reddit (`r/{TICKER}` + general subs) + named-account social only; no Seeking Alpha / Motley Fool / Zacks / aggregator op-eds.**

**Expected agent output:**

1. **Breaking news** — items published in the last 24–48 hours, each with named source + timestamp.
2. **Upcoming catalysts** — scheduled events in the next 7 days (earnings, FOMC, CPI, product launches, contract expirations, governance votes, etc.) with date/time.
3. **Whale / institutional activity** — large blocks, dark-pool prints, 13F changes, unusual options activity, on-chain transfers (for crypto).
4. **Protocol / corporate changes** — tokenomics changes, governance proposals, buybacks, splits, dilutions, M&A chatter.
5. **Regulatory developments** — agency actions, lawsuits, hearings, jurisdictional rule changes.
6. **24–48h price-moving read** — the agent's synthesis: which of the above is most likely to actually move the tape, ranked.
7. **Source list** — every claim links to a source; nothing assumed from memory.

**Reference output (example — $AMD):**

> Here's the deep-research, off-chart view for AMD focused on 24–48 hour price movers. I only used fresh web data and did not reuse any cached prices or odds.
>
> **🔥 Likely 24–48h catalysts**
>
> *1) BofA Global Technology Conference — today (June 2, 2026)*
> AMD's CFO Jean Hu is scheduled to present today. These conference talks can move the stock if guidance tone, margin outlook, AI roadmap, or supply commentary shifts expectations.
>
> *2) Export-control backdrop (sector-level)*
> A headline in the results notes U.S. tightening rules on AI chip exports to China (May 31, 2026). This can hit AMD and peers if analysts interpret it as demand constraint or margin pressure.
>
> **🧠 What's NOT showing up as a catalyst right now**
> - No fresh AMD-specific breaking news surfaced in the last 24h beyond the conference listing and standard IR/calendar entries.
> - No confirmed unusual options flow / whale activity from the accessible sources. I won't claim any without hard evidence.
>
> **📌 Upcoming but not immediate (outside 48h)**
> - Advancing AI 2026 event — July 23, 2026. Potentially large catalyst but outside your 48h window.
>
> **📉 Market-structure context (off-chart)**
> - One snapshot reported volume ~47.23M, about 81% of the 65-day average — not screaming "abnormal flow."
>
> **✅ What would actually move the price in 24–48h**
> - Conference remarks that meaningfully change AI / data-center revenue expectations.
> - Any new export-restriction commentary from U.S. agencies or geopolitical escalation.
> - Unexpected partnership / supply disclosure from AMD or a major OEM / cloud partner.
>
> If you want, I can:
> - Build a bull/bear catalyst map for AMD over the next 48 hours.
> - Run live unusual-activity checks from market data feeds (options volume, block trades) if you want that included.

---

## Prompt 4 — Trade Idea (Concrete Spec)

**Action:** Convert a thesis into a tradeable plan — fetch live price + orderbook depth, then return entry, target, stop loss, position size, and R:R. Include a same-window news scan to flag setup-invalidating events.
**Use when:** the user has converged on a name and wants the executable version of the idea.
**Scope:** any tradeable symbol. Platform-aware — the user names which data platform to source from (Futu, Webull, YahooFinance, etc.).
**Variables:** `{TICKER}` (e.g. `AMD`, `BTC`), `{PLATFORM}` (e.g. `Futu`, `Webull`, `YahooFinance`).

### 4.1 — Live trade plan with orderbook

**Template (user pastes, substitutes `{TICKER}` and `{PLATFORM}`):**

> Trade idea for {TICKER} on {PLATFORM}. Do not reuse any previously cached prices or odds from this conversation. Fetch live price and orderbook depth. Give me a specific entry, target, stop loss, position size, and risk/reward ratio. Search for any imminent news or events that could invalidate the setup — **sources per Source Policy: Reddit (`r/{TICKER}` + general subs) + primary; no aggregators.**

**Expected agent output:**

1. **Order book visualization** — fetched live via API and rendered as a depth table/chart.
   - ~40 levels around mid (20 ask / 20 bid default; adjust on request).
   - Columns: Price, Depth %, Size.
   - Ask side colored red above mid; bid side green below mid; mid line highlighted with current spread.
2. **Live price + book summary** — current mid, spread (absolute and %), total visible depth.
3. **Trade idea**:
   - Bias (long / short) and horizon.
   - **Entry** (limit / market / stop-buy) with exact price.
   - **Stop** (exact price).
   - **Target** (exact price).
   - **R:R** — show the math (Risk $, Reward $, ratio).
4. **Position size**:
   - Ask the user for their per-trade risk budget in USD if not provided.
   - Provide the sizing formula: `Shares = Risk $ / |Entry − Stop|`.
   - Offer 2 worked examples (e.g., $250 risk, $500 risk).
5. **Imminent news / event risk (next 24–48h)** — scan and explicitly state "none confirmed" when nothing surfaces; do not invent.
6. **Recalibration offer** — close by offering alternative setups (breakout entry, short side, tighter stop, higher R:R).

**Orderbook visualization reference:** screenshot example shows 40 levels around mid $506.29, spread $0.28 (0.055%), ask side (red) above mid running from $506.43 → $506.75, bid side (green) below mid running from $506.15 → $505.72, with Depth % and Size columns per level.

**Reference output (example — $AMD):**

> Here's a fresh, no-cache AMD (XYZ) trade idea with live price + order book, plus a quick catalyst check for the next 24–48 hours.
>
> **✅ Live price + order book (just fetched)**
> - Live price: $506.33
> - Order book depth: included in the chart above (40 levels)
>
> **📌 Trade idea (swing-long, one cycle ~19–22 trading days)**
> Bias: Long on pullback toward near-term support with defined risk.
>
> - Entry: $502.00 (limit)
> - Stop: $486.50
> - Target: $531.00
>
> **Risk/Reward**
> - Risk = 502.00 − 486.50 = $15.50
> - Reward = 531.00 − 502.00 = $29.00
> - R:R ≈ 1.87
>
> **Position size**
> Tell me your risk budget per trade in USD (e.g., $250 or $500) and I'll size it precisely.
> Formula: `Shares = Risk $ / 15.50`
>
> Example sizing:
> - $250 risk → ~16 shares
> - $500 risk → ~32 shares
>
> **🔍 Imminent news / event risk (next 24–48h)**
> The web scan did not surface any confirmed AMD-specific breaking events in the next 48 hours. The notable near-term factor is general US-China AI export restrictions, which remain an ongoing overhang but not a newly announced change in the last day.
>
> No confirmed new AMD press release, regulatory filing, or event within the next 48 hours surfaced in the latest search.
>
> If you want a different setup (breakout entry, short idea, tighter stop, higher R:R), I'll recalibrate instantly. Also, share your risk $ per trade and I'll lock the exact share size.

---

## Prompt 5 — Backtest Strategy

**Action:** Pick a reasonable strategy for the asset's character, fetch historical candles, run a backtest, and serve an interactive dashboard with equity curve, drawdown, and trade log.
**Use when:** the user wants to pressure-test a setup pattern or rule across history before committing capital.
**Scope:** any tradeable symbol with sufficient candle history. Platform-aware — user names the data source.
**Variables:** `{TICKER}` (e.g. `AMD`, `BTC`), `{PLATFORM}` (e.g. `Futu`, `Webull`, `YahooFinance`).

### 5.1 — Single-strategy backtest with dashboard

**Template (user pastes, substitutes `{TICKER}` and `{PLATFORM}`):**

> Backtest a simple strategy on {TICKER} on {PLATFORM}. Do not reuse any previously cached prices or odds from this conversation. Pick a reasonable strategy (momentum, mean reversion, or breakout depending on the asset's character), fetch historical candles, run the backtest, and serve an interactive dashboard with equity curve, drawdown, and trade log.

**Expected agent output:**

1. **Strategy chosen + rationale** — name it (e.g., "20/10 Donchian breakout, long-only") and one sentence on why it suits this asset's character.
2. **Data fetched** — instrument, timeframe (daily by default), lookback window.
3. **Backtest results** — structured block:
   - Start equity → End equity
   - Total return (%)
   - Max drawdown (%)
   - Number of trades
   - Win rate
   - Average hold (trading days)
4. **Interactive dashboard** — served as a standalone HTML page with:
   - Equity curve
   - Drawdown plot
   - Trade log (entry / exit / P&L / hold time)
   Link the user can open in a new tab.
5. **Recalibration offer** — "want a different style (mean-reversion / momentum) or longer lookback? I'll re-run."

**Reference output (example — $AMD):**

> Backtest complete — I used a Donchian breakout (20-day entry / 10-day exit, long-only) on AMD (XYZ) with daily candles and built an interactive dashboard showing equity, drawdown, and the trade log.
>
> **Key results (from the run):**
> - Start → End: $10,000 → $16,445.14
> - Total return: +64.45%
> - Max drawdown: −25.91%
> - Trades: 4
> - Win rate: 25%
> - Avg hold: 19.5 trading days
>
> Open standalone: AMD Backtest Dashboard
>
> If you want a different style (mean-reversion or momentum) or a longer lookback, I'll re-run it.

---

## Prompt 6 — Quick Summary

**Action:** One-line / one-screen view — price, day-over-day change, recent news.
**Use when:** the user wants a fast pulse-check on a name without full analysis.

_Templates to be added._
