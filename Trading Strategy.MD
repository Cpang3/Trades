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

---

## 2. Position sizing — anti-Martingale ±50%, tier-scaled (STRATEGY.md §4.1)

Base sizing varies by tier; within each tier the ±50% anti-Martingale modifier applies (size **up**
after a win, **down** after a loss).

| Tier | Base | After win (×1.5) | After loss (×0.5) | Hard ceiling | Hard floor |
|---|---|---|---|---|---|
| **Tier 1A** | 1.0% | 1.5% | 0.5% | 2.0% | 0.25% |
| **Tier 1B** | 0.5% | 0.75% | 0.25% | 1.0% | 0.25% |
| **Tier 1C** | 0.25% | 0.375% | 0.125% | 0.5% | 0.10% |

All percentages are **"% of account at risk per trade."**

- **Streak reset:** after 3 consecutive wins **or** 3 consecutive losses (any tier), sizing resets
  to the tier's base.
- **Rationale:** anti-Martingale is what professionals do. The opposite — doubling down after losses
  — is the classic account-killer and matches the user's stated "make it back" failure mode. Tier
  scaling further reduces exposure as the name's volatility/risk profile rises.

This is **Gate 7** of the pre-trade checklist. If the proposed contract premium × quantity exceeds
the tier's hard ceiling → **SKIP** or use shares instead.

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
