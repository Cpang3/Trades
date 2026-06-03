# Trading Strategy — Sizing, P&L & Instrument Selection

> **Compartmentalized from `STRATEGY.md` v0.3 §2 (style), §4.1–4.3 (sizing / withdrawal / full
> exit), §5.2 (Bot 2 screener), §6 (exit waterfall), §8 (shares vs options).**
> This file covers **how big, when to take profit/cut, and which instrument** — the mechanics that
> turn a graded setup into a sized, exit-planned position.
>
> Part of a 5-file set: `watchlist.MD` · `Trading Rules.MD` · `Macro Indicators.MD` · `Technical Analysis.MD` · `Trading Strategy.MD`

---

## 1. Style & horizon (STRATEGY.md §2)

**Regime-confirmed swing trading on the daily chart**, expressed as long shares or long calls/puts.

| Element | Value |
|---|---|
| Signal lookback (Markov) | 252 trading days |
| Regime window (Markov) | 20 days |
| Option DTE at entry | 30–60 DTE |
| Average hold | 2–15 trading days |
| Max hold | No hard limit; ruled by the exit waterfall (§5) |

**No LEAPs** sized off the 20-day regime signal — horizon mismatch.

### Preferred setup style — ranked

When multiple A-grade setups compete, prefer in this order:

1. **Momentum breakout** — breakout from a tight base or flag, with volume expansion. Captures the trend's first leg; cleanest invalidation (close below the pivot).
2. **Trend-following swing** — long the established uptrend; entries on pullbacks to 20/50 EMA or higher-low continuation.
3. **Mean reversion pullback** — retest of a prior breakout level; wait for the failed-rejection re-entry, not the first touch.

The ranking is expressed quantitatively in §7 via a screener score multiplier (×1.10 / ×1.00 / ×0.90).

---

## 2. Position sizing, leverage & streak rules (STRATEGY.md §4.1)

### 2.1 Position sizing — anti-Martingale ±50%, tier-scaled, with A+ escalator

Base sizing varies by tier; within each tier the ±50% anti-Martingale modifier applies (size **up**
after a win, **down** after a loss). An **A+ escalator** doubles the next size after a trade hits
its **full** profit target — rewarding trades that ran the full distance, not just a partial.

| Tier | Base | Standard win (×1.5) | A+ win (×2.0) | After loss (×0.5) | Hard ceiling | Hard floor |
|---|---|---|---|---|---|---|
| **Tier 1A** | 1.0% | 1.5% | **2.0%** | 0.5% | 2.0% | 0.25% |
| **Tier 1B** | 0.5% | 0.75% | **1.0%** | 0.25% | 1.0% | 0.25% |
| **Tier 1C** | 0.25% | 0.375% | **0.5%** | 0.125% | 0.5% | 0.10% |

All percentages are **"% of account at risk per trade."**

**Outcome definitions:**
- **Standard win** = trade closed positive but did not run the full target (e.g. partial target hit, runner exited at breakeven or modest gain after stop trail).
- **A+ win** = trade closed at the **full** profit target per §5.3 (e.g. +40% capital / +100% premium for Tier 1A). The system rewards trades that ran the full distance.
- **Loss** = closed below entry, regardless of mechanism (chart stop, thesis-broken exit per §5.1, time stop per §5.5).

**Hard rule (preserved — non-negotiable):** the bot **never** averages down into a losing trade.
A failed thesis is closed under §5.1, not added to. Doubling down after losses is the classic
account-killer and matches the user's stated "make it back" failure mode. The 8-gate engine
rejects any proposal to add to a position below entry.

### 2.2 Leverage — tier-gated

Leverage is permitted only on Tier 1A and Tier 1B names; Tier 1C is unleveraged.

| Tier | Default leverage | Max leverage | Notes |
|---|---|---|---|
| **Tier 1A** | 2x | 3x | Mega-cap, deepest liquidity, tightest spreads — supports leveraged exposure |
| **Tier 1B** | 1.5x | 1.5x | Moderate; restricted by the tier's volatility profile |
| **Tier 1C** | 1x (none) | 1x (none) | Volatility too high; leverage compounds the failure mode |

**Application rules:**
- "Leverage" here means **margin on shares**. Options carry implicit leverage via delta; the multipliers above are **shares-only** unless explicitly extended.
- Leverage does **not** change the §2.1 risk denominator. "% of account at risk" still refers to the **dollar loss** from entry → stop, regardless of notional deployed. Leverage permits more notional exposure for the same dollar risk.
- Stops remain chart-based per §5. Leverage **cannot widen a stop**. If the chart-based stop sized at full leverage would exceed the tier's hard ceiling at risk → **reduce position size or skip the trade**. Never loosen the stop to fit the leverage.

### 2.3 Stop-loss default

Every entry **must** be paired with a stop (preserved hard rule from `Trading Rules.MD §1`).

- **Preferred:** chart-based stop (prior swing low / S-R / EMA shelf). The user identifies the level on the chart and supplies it explicitly.
- **Default fallback:** if no chart-based level is supplied, the bot uses **5% below cost-basis** for longs (and 5% above for shorts when shorting is later enabled). This is the failsafe — explicit chart stops are still preferred.

The broker layer (`tradingbot broker-place ... --stop`) requires the stop price; the CLI also accepts `--stop-pct` (default 5.0) which derives the stop from `--entry`.

### 2.4 Streak rules and cooldowns

- **3 consecutive wins** (any combination of standard / A+) → next trade resets to **tier base**. Prevents runaway sizing on a hot streak.
- **3 consecutive losses** → next trade resets to **tier base AND triggers a hard cooldown** per `Trading Rules.MD §4` (3 trading days default; extendable to 5 days to span the weekend for emotional reset). The cooldown blocks all new entries; open positions are managed under §5 normally.

This is **Gate 7** of the pre-trade checklist. If the proposed contract premium × quantity
exceeds the tier's hard ceiling → **SKIP** or use shares instead.

---

## 3. Withdrawal discipline — 50% profit → 50% withdrawal (STRATEGY.md §4.2)

When a **closed** trade returns **> +50% on premium** (options) or **> +20% on capital** (shares):

- **50% of the realized profit** is mechanically flagged for withdrawal from the brokerage.
- The bot's "available risk capital" excludes withdrawn funds from that point forward.
- **Monthly sweep:** at month-end the bot reports `withdraw_eligible: £X` and the user transfers it
  out.

**Rationale:** removes capital from the risk pool as a function of *realized* wins, not unrealized
paper gains. Breaks the "I'm up, let me push more" dopamine loop and compounds the cash sleeve over
time — not just the trading account. **This is the single most important discipline rule for the
user.** The advisor reminds the user every time a trade clears the +50% / +20% threshold.

---

## 4. 100% gain → full exit, redeploy (STRATEGY.md §4.3)

If a position reaches **+100% on premium** (options) or **+40% on capital** (shares):

- **Full exit.** Close 100% of the position.
- Redeploy capital with revised sizing per §2.

**Rationale:** user's stated practice. Letting +100% winners run risks giving back realized gains
for tail outcomes the user does not historically capture. Lock the 100%, redeploy, repeat. (A single
+100% winner also triggers a 24h cooldown — see `Trading Rules.MD` §4.)

---

## 5. Exit waterfall (STRATEGY.md §6)

**Order matters. The first condition that fires wins. No discretion overrides.**

| # | Condition | Action |
|---|---|---|
| 6.1 | **Thesis broken** — support breaks, flag fails, regime flips | **Close immediately.** Don't wait for the stop. |
| 6.2 | **Stop-loss** — chart stop touched | Close. *(Options: also close if premium has declined ≥ 50% from entry, even if the chart stop isn't hit — the optionality has decayed.)* |
| 6.3 | **Profit target hit** (tier-scaled, see below) | Partial: take 50% off, move remainder stop to breakeven. Full: exit 100%, redeploy per §2. |
| 6.4 | **Pre-market / active-window favorable move** | Shares: gap/move ≥ 10% in favor → recommend exit/partial. Options: premium ≥ 50% in favor → recommend scale. |
| 6.5 | **Time stop (options)** | At DTE ≤ 21: if at any loss → close. If in profit → evaluate roll (close current, open further-dated). Shares: no time stop. |
| 6.6 | **Delta limit (options)** | If delta > 0.85, the contract is a stock proxy with theta drag — consider closing and re-entering lower-strike / longer-dated. |

### Profit targets — tier-scaled (STRATEGY.md §6.3)

Targets tighten as tier risk rises (the right tail is less reliable on higher-vol / lower-quality
names, so take profit faster).

| Tier | Partial target (take 50% off) | Full exit |
|---|---|---|
| **Tier 1A** | +50% premium / +20% capital | +100% premium / +40% capital |
| **Tier 1B** | +30% premium / +15% capital | +75% premium / +30% capital |
| **Tier 1C** | +25% premium / +10% capital | +50% premium / +20% capital |

At the **partial** target: take 50% off, move the remainder's stop to breakeven.
At the **full** target: full exit per §4, redeploy with revised sizing per §2.

### Pre-market & active-window exits **[user modification of EMS]** (STRATEGY.md §6.4)

Allowed on existing positions in any non-passive window — HK morning pre-market, preferred entry
window (HK 21:30–23:00), and management-only window (HK 23:00–01:00). Standing GTC orders cover the
passive window. **Pre-market entries remain forbidden** — exits and standing-order adjustments only.

---

## 6. Shares vs options selection (STRATEGY.md §8) — Gate 8

When the screener says ENTER, which instrument?

| Condition | Choose |
|---|---|
| Bot 1 mode = `LONG_STOCKS_OK` **AND** IV rank ≥ 50 | **Shares** |
| Bot 1 mode = `LONG_OPTIONS_PREFERRED` | **Options** |
| Bot 1 mode = `LONG_STOCKS_OK` **AND** IV rank < 30 | **Options** (cheap convexity available) |
| Earnings inside DTE | **Options** (defined risk required) |
| Single ATM contract premium > 2% of account | **Shares OR pass** |
| Otherwise (default) | **Options** at 30–60 DTE, delta 0.30–0.50, single-leg long call/put |

**Account-size reality:** at ~£10K, many Tier 1 options won't fit the 1–2% sizing rule. The sizing
policy will sometimes mean "no trade today — no contract fits the budget." **That is correct
behavior.**

---

## 7. Bot 2 — Name screener (STRATEGY.md §5.2)

**Input:** a specific watchlist name + the current mode from Bot 1 + account state (cooldown?
drawdown?).
**Output:** weighted score 0–100 + letter grade + ENTER / WAIT / SKIP + any failure flags.

### Weighted subsections (sum to 100%)

| Subsection | Weight | Measures |
|---|---|---|
| Relative Strength (60d vs SPY) | 20% | Leadership in the current tape |
| Base Structure / Tightness | 20% | VCP-style base, depth, contraction |
| Pivot / Pattern Behavior | 20% | EMS S/R + flag/wedge detection on daily |
| Volume Behavior | 10% | Dry-up in base + expansion on breakout |
| Macro Regime Fit | 10% | Alignment with Bot 1 mode |
| IV / Options Suitability | 10% | IV rank, OI, bid-ask, contract liquidity |
| Earnings Proximity | 5% | DTE-relative; binary catalyst flag |
| Institutional / Quality | 5% | Mcap, ADV, fund holdings (informational) |

*(Technical structure = 60%: RS 20 + Base 20 + Pattern 20.)*

### Style multiplier (applied to final score)

After the weighted score is computed, multiply by the style modifier derived from the EMS pattern detector. This expresses the §1 preferred-style ranking quantitatively.

| Setup style | Multiplier | Detected from |
|---|---|---|
| **Momentum breakout** | ×1.10 | `flag_breakout`, breakout from a tight base (VCP-style) |
| **Trend-following swing** | ×1.00 | Pullback to 20/50 EMA, higher-low continuation |
| **Mean reversion pullback** | ×0.90 | `pivot_reclaim`, retest of a prior breakout level, oversold bounce into S/R |

The multiplier nudges momentum setups toward higher grade bands and mean-reversion setups
toward lower bands without changing the underlying weights. Failure flags below still cap the
post-multiplier score.

### Failure flags (cap the grade regardless of subsection scores)

| Flag | Cap |
|---|---|
| RS rating < 30 | C max |
| Mode = `DEFENSIVE_FLAT` | C max |
| Account drawdown ≥ −5% AND earnings inside DTE | D max |
| Underlying outside Tier 1 universe | **SKIP** |
| Account on cooldown | **SKIP** |
| Daily loss limit hit | **SKIP all today** |
| No A+ setup (no S/R interaction AND no flag/wedge) | D max |
| Volume not confirming | C max |

### Output mapping

| Score | Grade | Action |
|---|---|---|
| 80–100 | A | ENTER, full size per §2 |
| 65–79 | B | ENTER, 0.5× size |
| 50–64 | C | WAIT — flagged with the missing condition |
| 35–49 | D | SKIP, log reason |
| 0–34 | F | SKIP, reviewed at next weekly watchlist update |

> **Tier overlay:** Tier 1B and Tier 1C require **grade A only** — no B-grade entries — on top of
> their half / quarter sizing. Tier 1A may enter on A or B.

Sample output line:

> `NVDA: B (72/100) — ENTER at 0.5x size. Missing: pivot reclaim not yet confirmed; flag breakout
> pending volume expansion.`

---

## 8. Every ENTER must state (the trade is not a trade without these)

Per the buying-vs-trading rule (`Trading Rules.MD` §1), an ENTER recommendation must include:

- **Entry trigger** — limit price + condition
- **Stop** — chart-based level + reasoning (Tier 1C: AND −25% premium, whichever fires first)
- **Partial target** — tier-scaled (§5.3)
- **Full target** — tier-scaled (§5.3)
- **Time stop (options)** — DTE ≤ 21 trigger
- **Thesis-break trigger** — what invalidates the trade (e.g., "NVDA closes below $180 daily → exit")

If any of these is undefined, it is a **buy proposal, not a trade** — reject until the exit plan is
complete.
