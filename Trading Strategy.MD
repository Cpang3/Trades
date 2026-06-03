# Trading Strategy — Sizing, P&L & Instrument Selection

> **Compartmentalized from `STRATEGY.md` v0.3 §2 (style), §4.1–4.3 (sizing / withdrawal / full
> exit), §5.2 (Bot 2 screener), §6 (exit waterfall), §8 (shares vs options).**
> This file covers **how big, when to take profit/cut, and which instrument** — the mechanics that
> turn a graded setup into a sized, exit-planned position.
>
> Part of a 5-file set: `watchlist.MD` · `Trading Rules.MD` · `Macro Indicators.MD` · `Technical Analysis.MD` · `Trading Strategy.MD`

---

## 1. Style & horizon (STRATEGY.md §2) — TWO FLOWS (revised 2026-06-03)

The bot operates two distinct flows with different sizing, stop, and exit rules. Each prompt and CLI command targets one flow explicitly.

### 1.A Swing flow (default, primary)

**Regime-confirmed swing trading on the daily chart**, held one monthly options cycle (~19–22 trading days). Expressed as long shares or long calls/puts.
- **Templates:** `Prompts.MD §1.1` (25 templates)
- **Stops:** 5% default fallback OR chart-based (preferred); see §2.3
- **Sizing cap:** max-loss cap (Tier 1A 4%, 1B 2%, 1C 0.5%) — §2.1
- **DCA:** up to 3 averages allowed on Tier 1A/1B; Tier 1C no-DCA
- **Exit waterfall:** 4H MA / VWAP / max-loss / DCA-exhausted (§5)

### 1.B Day-trade flow

**Intraday momentum / mean-reversion**, held minutes-to-hours, closed same session.
- **Templates:** `Prompts.MD §1.2` (15 templates)
- **Stops:** tight, 1-2% from entry (intraday structure)
- **Sizing:** tier-base ceiling from §2.1 (1A 1.0%, 1B 0.5%, 1C 0.25%)
- **No DCA** (insufficient time intraday)
- **Time stop:** close by HK 00:00 / US 12:00 EDT if not at target

The two flows are **non-overlapping** — a given trade is one or the other, never both. The prompt invoked picks the flow.

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

**Within-trade DCA — controlled averaging up to 3 times (REVISED 2026-06-03)**

The earlier "never average down" hard rule has been replaced by a controlled, capped DCA mechanism. The user explicitly chose to allow averaging because the fixed-% ceiling produced unworkable position sizes on small accounts AND because chart-structure-based exits give a more honest read on when a thesis is actually broken.

**DCA rules:**

- A position may be added to **up to 3 times** after the initial entry (max 4 tranches total)
- Each DCA must be at a **worse price** than the prior tranche (true averaging down)
- Each DCA is sized the **same per-share-risk** as the initial entry — no escalating dose
- **DCA is allowed only on Tier 1A and 1B names**; Tier 1C remains no-DCA (the failure-mode list — AIP/SRPT/WULF/BMMR — were all sub-$1B narrative names; the discipline stays tight there)
- DCAs are tracked in the journal (`dca_count` per open trade)

**Mandatory cut conditions (any one fires → close the full position):**

1. **4-hour MA breach** — daily close below the 20-period MA on 4-hour candles
2. **4-hour VWAP breach** — daily close below daily-anchored VWAP on 4-hour candles
3. **Per-trade max-loss cap** — cumulative trade loss hits the tier's max-loss threshold (see table below)
4. **DCA-exhausted fail-safe** — price down ≥ 5% from average cost basis AFTER the 3rd DCA has been placed

**Per-trade max-loss cap (the absolute backstop):**

The fixed-$ position-size determinant is replaced by a **max-loss-cap** that limits the worst-case outcome including all DCAs. Set at 2× the previous "ceiling at risk" to accommodate the up-to-4-tranche position growth:

| Tier | Single-tranche risk (initial) | Max-loss cap (after DCAs, fail-safe) |
|---|---|---|
| **Tier 1A** | 1.0% (base) up to 2.0% (escalated) | **4.0%** of equity |
| **Tier 1B** | 0.5% / 1.0% | **2.0%** of equity |
| **Tier 1C** | 0.25% / 0.5% (no DCA allowed) | **0.5%** of equity (unchanged) |

**Why this rule was reversed (transparency):**

The user's stated historical failure mode in `Trading Rules.MD §0` was "full-port concentration on speculative names" (AIP/SRPT/WULF/BMMR). The earlier draft kept "no DCA" as the hard rule to protect against that pattern. The reversal is deliberate — the user weighed it explicitly — and the new framework defends against the failure mode via THREE different cap mechanisms (DCA count cap, structure-based exits, max-loss cap), not by forbidding the technique outright. **If you find yourself wanting to DCA a 4th time, the system says no.** That's the discipline.

**Hard rule (still preserved):** Tier 1C is no-DCA, period. Quarter sizing + grade-A-only + tight stops + 90-day suspension on 3 consecutive 1C losses are the existing layered protections for the failure-mode tier; we don't loosen them.

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
| 6.2a | **4-hour MA breach** — daily close below 20-period MA on 4H candles | **Close.** Primary chart-structure exit (revised 2026-06-03). |
| 6.2b | **4-hour VWAP breach** — daily close below daily-anchored VWAP on 4H candles | **Close.** Confirms 6.2a from a volume-weighted lens. |
| 6.2c | **Per-trade max-loss cap hit** (per §2.1 table: 1A 4%, 1B 2%, 1C 0.5% of equity) | **Close.** Hard fail-safe regardless of chart. |
| 6.2d | **DCA-exhausted fail-safe** — down ≥ 5% from avg cost basis after 3rd DCA | **Close.** Discipline cap: 3 DCAs is the limit; if still wrong, out. |
| 6.2e | **Stop-loss** — chart stop touched (legacy explicit stop) | Close. *(Options: also close if premium has declined ≥ 50% from entry, even if the chart stop isn't hit — the optionality has decayed.)* |
| 6.3 | **Profit target hit** (tier-scaled, see below) | Partial: take 50% off, move remainder stop to breakeven. Full: exit 100%, redeploy per §2. |
| 6.4 | **Pre-market / active-window favorable move** | Shares: gap/move ≥ 10% in favor → recommend exit/partial. Options: premium ≥ 50% in favor → recommend scale. |
| 6.5 | **Time stop (options)** | At DTE ≤ 21: if at any loss → close. If in profit → evaluate roll (close current, open further-dated). Shares: no time stop. |
| 6.6 | **Delta limit (options)** | If delta > 0.85, the contract is a stock proxy with theta drag — consider closing and re-entering lower-strike / longer-dated. |
| 6.7 | **DCA-add trigger** (not an exit; documented here for completeness) | If thesis intact AND price below entry by tier-defined increments AND `dca_count < 3` AND on Tier 1A/1B: optionally add a DCA tranche at the same per-share risk as the initial (see §2.1). |

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
