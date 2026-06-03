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

### 1.1 — Watchlist-scoped one-cycle SWING ideas (Stocks)

**Use when:** the user is out of original trade ideas and wants a templified list of candidates drawn from the existing watchlist.
**Scope:** stocks listed in `watchlist.MD` (this repo) / `watchlist.yaml` (tradingbot repo) only — the agent should NOT pull from the broader market.
**Horizon:** one standard monthly options cycle (~19–22 trading days, ~21 on average).

**Default output shape — 9 stocks, full-spectrum tier coverage, ranked by composite score:**

- **3 from Tier 1A** (mega-cap quality — NVDA, AMD, GOOGL, META, AAPL, MSFT, AMZN, TSLA, SPY, QQQ)
- **3 from Tier 1B** (narrative growth, auto-pass numeric filters)
- **3 from Tier 1C** (override candidates — top 3 of the moment, each respecting `watchlist.MD` §2 override discipline)

The user wants the full opportunity surface, not just the safest end. The 9-name list gives 3-deep coverage of each tier; the composite score lets the user see *which 1B beats which 1A* at any given moment.

#### Composite scoring — 60% quant / 40% qual (0–100 scale)

**Composite = 0.60 × Quant + 0.40 × Qual**

Weighting choice: rules-based (quant) carries the heavier weight because the user's stated failure mode is emotional override — anchoring more of the rank in §7's screener output protects against the "I like this name" drift. Qualitative still matters because catalysts, narrative momentum, and setup cleanliness aren't fully captured numerically.

**Quant Score (0–100)** — fully rules-based, derived from `Trading Strategy.MD §7`:

| Component | Max points | Source |
|---|---|---|
| Screener grade (post-style-multiplier) | 60 | §7 screener: A = 60, B = 35, C = 10 (no C/D should reach this list) |
| R:R for the one-cycle hold | 20 | ≥3.0 = 20 · 2.5 = 16 · 2.0 = 12 · 1.5 = 8 · <1.5 = 0 |
| Distance-to-invalidation (entry → stop, % of price) | 10 | tighter = better; ≤3% = 10, 3–5% = 7, 5–8% = 4, >8% = 0 |
| Liquidity headroom (ADV vs typical position) | 10 | comfortable = 10, adequate = 5, tight = 0 |

**Qual Score (0–100)** — agent's structured judgment across four lenses:

| Component | Max points | What the agent is reading |
|---|---|---|
| Catalyst quality (next 19–22 days) | 30 | Upcoming earnings / product / macro inside the window — favorable, neutral, or hostile to the thesis |
| Narrative momentum | 25 | Is the story building or fading? Per Source Policy: Reddit ticker sub + general subs + named-account social |
| Setup cleanliness | 25 | Subjective TA read — does the chart look like an A+ setup, or is it a B that the screener over-rewarded? |
| Conviction qualifier | 20 | Is this one of the best trades on the watchlist right now, or just the best in its tier? |

#### Ranking rule

The 9 ideas are ordered **top → bottom by Composite Score**, across tiers. Rank 1 is the most actionable trade right now — could be a 1A, 1B, or 1C. Rank 9 is still tradeable but lower priority.

Each idea includes: thesis, setup style + multiplier, trigger, invalidation, one-cycle target, R:R, Quant / Qual / Composite scores.

**Override:** if the user explicitly specifies a different count or tier mix in their prompt (e.g., "give me 3 ideas from Tier 1A only", "show 10 shorts", "skip Tier 1C this week"), the override wins. With no override, return 9 stocks 3/3/3 ranked by composite.

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

### 1.2 — Day Trade Ideas (intraday close)

**Use when:** the user wants intraday momentum / mean-reversion ideas with same-session close (entered after US open, closed before HK 01:00 / US 13:00 EDT). Distinct from §1.1 swing flow.

**Scope:**
- Watchlist names only (same Tier 1A/1B/1C universe as §1.1)
- Held intraday — typically minutes to a few hours
- Exit before HK 01:00 (after that, user is asleep — no day-trade discretion possible)
- Setups read from intraday charts (5m/15m/1h candles)

**Sizing & risk (distinct from §1.1 swing):**
- Stops are tight: **1-2% from entry** (intraday structure: prior 15m low / VWAP / opening range break)
- Tier-base sizing applies as defined in `Trading Strategy.MD §2.1`:
  - Tier 1A day trade: 1.0% base risk per trade (= 1.5% standard win / 2.0% A+ win / 0.5% loss-modified)
  - Tier 1B day trade: 0.5% base
  - Tier 1C day trade: 0.25% base (Tier 1C is high-vol — day trades carry extra friction; consider grade-A-only requirement)
- The tier-base ceiling fits the 1-2% stops mechanically:
  - At 1A: $76.77 × 1% = $0.77 risk; at 1.5% stop = $51 position size
  - Math works because stops are tighter, NOT because the ceiling is looser

**No DCA on day trades.** DCA is a swing-trade mechanism (§2.1) — there isn't time intraday for the thesis to play out across multiple averages. If you're stopped, you're stopped.

**Templates the user can paste/select (15 day-trade prompts):**

1. "Give me 3 day-trade momentum setups in U.S. mega-caps for entry at the open (HK 21:30 EDT 09:30)."
2. "Find pre-market gappers up 3-15% (NOT runners above 20%) with positive news, suitable for intraday continuation."
3. "Identify 3 VWAP-bounce setups: mega-cap names currently dipping toward VWAP after morning push."
4. "Show 5 watchlist names with opening-range breakout potential — 15m range tight, volume building."
5. "Find 3 intraday short setups: stocks failing prior-day high with weak relative strength."
6. "Give me 3 reversal setups in oversold mega-caps after morning weakness (RSI < 30 on 5m, S/R reclaim signal)."
7. "Scan for stocks holding above the 1-hour 20-EMA with declining volatility — intraday trend-continuation candidates."
8. "Find pullback-to-VWAP setups for intraday entry, entry zone within 1% of VWAP, stop tight at -1.5%."
9. "Identify 3 day-trade candidates around catalyst windows (CPI / FOMC release minus 2 hours / earnings AH the night before)."
10. "Give me 2 mean-reversion intraday plays: oversold bounce candidates with clear S/R reclaim."
11. "Scan for stocks with relative-strength leadership today (% green vs. sector ETF) — momentum-continuation day trades."
12. "Find 3 intraday breakout setups in defense / drone names with positive catalyst flow."
13. "Show 5 stocks failing 5m support that could be short-on-failed-rally intraday."
14. "Give me 3 day-trade options plays in mega-caps — 0DTE or weekly, delta 0.30-0.50, IV-rank constraint applied per §7 screener."
15. "Find candidates for the 'first 30-minute high break': stocks that pushed early then consolidated below the 9:30-10:00 high."

**Default output (per ticker):**
- Bias (long / short)
- Trigger (specific intraday level — e.g. "close > 5m high after 09:45")
- Stop (1-2% from entry — intraday chart-based)
- Target (1-2R or VWAP / prior session high, whichever is closer)
- Time stop (close by HK 00:00 / US 12:00 if not at target — avoid lunch lull)
- Tier and sizing per `Trading Strategy.MD §2.1`

**Day-trade discipline carry-overs from `Trading Rules.MD`:**
- Capital efficiency window (Mon-Thurs only — §3)
- Macro/event filter (no entries within 48h of FOMC/CPI/PPI — §5 #3)
- Pre-market chase guard (no buying names up ≥20% pre-market — §5 #7 revised)
- Cooldowns + streak rules apply identically

**Reality check on small accounts:** day trading at ≤ $1K real is fee-prohibitive (Futu ~$0.99 per fill × 2 = $1.98 round-trip = 5-20% of a single-share position on the watchlist). Day-trade flow is designed but defer real-money execution until the account is >$5K. Paper-test the flow first.

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

## Prompt 6 — RESERVED

**Status:** Reserved slot. Original "Quick Summary" framing dropped; will be repurposed for a different action TBD.

_Do not fill until the new action is defined._

---

## Prompt 7 — Watchlist Expansion

**Action:** Add one or more new tickers to the watchlist universe, with full metadata, and confirm every downstream registry (Reddit URL registry, screener, etc.) picks up the new entries.

**Use when:** the user wants to expand the universe — typically after research surfaces a new setup, sector rotation, or theme.

**Scope of changes per ticker:**

| File | Repo | What gets updated |
|---|---|---|
| `watchlist.yaml` | tradingbot | Append entry with full required fields. CLI reads this on every invocation. |
| `watchlist.MD` | Trades | Add a row in the appropriate theme table (human-readable mirror). |
| `reddit_overrides.yaml` | tradingbot | Only if the ticker's real retail community sub differs from the naive `r/{TICKER}` (e.g. AMD → `r/AMD_Stock`). |

**Variables:**

- `{TICKER}` — e.g. `MRVL`
- `{TIER}` — `1A` / `1B` / `1C` per `watchlist.MD §1`
- `{THEME}` — one of: `semi_ai`, `photonics`, `ai_software`, `crypto`, `space`, `defense`, `fintech`, `energy`, `hardware`, `healthcare`, `industrial`
- `{THESIS}` — 1-2 sentence thesis
- `{EARNINGS_DATE}` (optional) — ISO `YYYY-MM-DD`
- `{PATTERNS_TO_WATCH}` (optional) — list e.g. `[flag_breakout, support_hold]`

### 7.1 — Add new tickers (and propagate to all registries)

**Template (user pastes, fills in tickers):**

> Add the following ticker(s) to my watchlist:
>
> - Ticker: `{TICKER1}`, Tier: `{TIER}`, Theme: `{THEME}`, Thesis: `{THESIS}`
> - Ticker: `{TICKER2}`, ...
>
> Update both `tradingbot/watchlist.yaml` and `Trades/watchlist.MD`. For each ticker, decide whether the naive `r/{TICKER}` subreddit is the actual retail community; if not (e.g. `r/AMD_Stock` instead of `r/AMD`), add the correct override to `tradingbot/reddit_overrides.yaml`. After updating, verify the Reddit URL registry rebuilds for each new ticker by running `tradingbot reddit-urls {TICKER}`. Commit and push both repos.

**Expected agent output:**

1. **Validation pass** — confirm tier is `1A`/`1B`/`1C`, theme is in the canonical list, ticker isn't already on the watchlist.
2. **Tier 1C discipline check** — if any ticker is `1C`, require `override_reason` (which numeric filter is being bypassed) and `failure_mode_check` (comparison to historical-loss tickers per `watchlist.MD §2`). Refuse to add the 1C without these.
3. **Updates to `watchlist.yaml`** — append entries with all required fields.
4. **Updates to `watchlist.MD`** — add rows in the appropriate theme table; update the tier counts in the header.
5. **Reddit override decision** — for each new ticker, the agent reasons about whether `r/{TICKER}` is the real community. If clearly wrong (e.g. `r/AMD` doesn't track AMD), add an override. If unsure, surface the URL for the user to verify.
6. **Verification run** — `tradingbot reddit-urls <NEW_TICKER>` to confirm URLs render.
7. **Push** — commit both repos with descriptive messages referencing this prompt.

**Iteration:** repeat this prompt any time the watchlist grows. Downstream commands (`screen-all`, `reddit-urls`, etc.) auto-pick-up new entries — they re-read `watchlist.yaml` on every invocation, so no separate registry rebuild is needed.

**Reference output (example — adding hypothetical `XYZ`):**

> Validated:
> - XYZ (Tier 1B, theme `ai_software`) — not already on watchlist ✓
>
> Updated:
> - `tradingbot/watchlist.yaml` — appended entry
> - `Trades/watchlist.MD` — added row in "AI software / cloud / data (14)" table; bumped header count from 13 → 14
> - `tradingbot/reddit_overrides.yaml` — no override added (naive `r/XYZ` looks correct on inspection)
>
> Verification:
> ```
> [1B] XYZ (ai_software)
>   https://www.reddit.com/r/XYZ
> ```
>
> Pushed:
> - Trades@abc1234
> - tradingbot@def5678
