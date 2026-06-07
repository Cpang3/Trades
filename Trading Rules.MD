# Trading Rules

> **Compartmentalized from `STRATEGY.md` v0.3 §0.1, §1, §3.9–3.10, §4.4, §4.6, §9, §10**
> plus the swing-skill 8-gate pre-trade checklist.
> This is the **discipline / governance** layer: when am I allowed to trade, how do I not blow up,
> and what the bot refuses to execute. The bot is the impulse-control layer — these are the rules
> it enforces regardless of how the user feels on the day.
>
> Part of a 5-file set: `watchlist.MD` · `Trading Rules.MD` · `Macro Indicators.MD` · `Technical Analysis.MD` · `Trading Strategy.MD`

---

## 0. Why this file exists

The user has a diagnosed history of emotional override under pressure — revenge trading after
losses, earnings-binary gambling, rotating winners too early, full-port concentration on
speculative names (historical losses: AIP / SRPT / WULF / BMMR). The edge (deep fundamental +
narrative research) is real; it gets undermined by emotion. **The bot is the override.** If the bot
says SKIP, the user SKIPs. A skipped trade is never a problem; a rule-broken trade always is.

---

## 1. Foundational distinction: buying vs trading (STRATEGY.md §0.1)

This distinction is the philosophical basis for every exit rule. Hold it in mind always.

> **Buying a stock because you like it = investing without a plan.** No exit, no horizon, no
> realized return. It's an opinion held in the brokerage account. It can move 100% either way and
> the holder may do nothing, because there was never a plan.
>
> **Trading = positioning + a defined exit.** Every position has a planned exit *at entry*. You
> realize gains only by selling. The exit is what makes you a trader, not a passive holder.

**Every entry must be paired with all of:**
- A chart-based stop
- A partial profit target
- A full exit target
- A thesis-break invalidation level
- A time stop (on options)
- The 50% withdrawal mechanic on big winners
- A capital efficiency window for forcing turnover

**If a trade is proposed without all of these defined, it is not yet a trade — it is a buy
proposal. Reject until the exit plan is complete.** (Non-negotiable.)

---

## 2. The Hong Kong execution model (STRATEGY.md §1)

The user lives in Hong Kong and holds a day job at FDT. US hours (09:30–16:00 EDT) map to
~21:30–04:00 HK. The user is available for live US-session activity until **~01:00 HK**, covering
roughly the **first 3.5 hours of the US session — the entire US morning.** After HK 01:00 the user
is asleep; the rest of the session runs on standing orders only.

### Two-window model

| Window | HK time | US time (EDT) | What happens |
|---|---|---|---|
| **Prep** | 06:00–21:00 HK | overnight US → pre-market | Bot 2 runs the overnight screener on yesterday's close. HK morning: review plan, finalize watchlist, place limit entries, set GTC stops + targets. FDT day job 11:00–21:00. |
| **Active** | 21:00–01:00 HK (~3.5h) | 09:00–13:00 EDT | US opens 09:30 (HK 21:30). Watch the open live, react, take fired targets, enter on real-time A+ setups inside the morning window. |
| **Passive** | 01:00–04:00 HK | 13:00–16:00 EDT | User asleep. Standing limit/stop/GTC orders rest. No discretion. The plan as it stood at HK 01:00 finishes the session. |

### Active-window sub-windows

| Sub-window | HK time | US time | Permitted |
|---|---|---|---|
| **Preferred entry** | 21:30–23:00 HK | 09:30–11:00 EDT | New entries on A+ setups. Highest edge. |
| **Management-only** | 23:00–01:00 HK | 11:00–13:00 EDT | **No new entries** (lunch lull). Take profit targets, adjust stops only if structure changes, exit on thesis breaks. |
| **Passive** | after 01:00 HK | 13:00–16:00 EDT | No user activity. Standing orders only. |

**Implications baked in:**
- Entries placed as limit orders in the prep window; allowed live in the preferred entry window on a
  fresh A+ setup.
- Any exit target that fires in the active window — the user acts. Targets firing in the passive
  window are caught by standing GTC orders.
- **Same-day round-trips — flow-dependent (revised 2026-06-03).**
  - **Swing flow (`Prompts.MD §1.1`, default):** no same-day round-trips. Holds remain minimum overnight per the original timezone + swing-style logic.
  - **Day-trade flow (`Prompts.MD §1.2`):** same-day round-trips allowed by design — that's the point. Day trades close before HK 01:00 / US 13:00 EDT (when the user goes to sleep). The two flows are non-overlapping; a given trade is one or the other.
  - **Fee/economics reality check:** day trading at small account sizes is fee-prohibitive (Futu commission ~$0.99/fill × 2 = $1.98 round-trip = ≥5% of a 1-share position on most watchlist names). Paper test the flow; defer real-money day trades until account >$5K.
- **All orders are limit orders** unless taking a market exit on a triggered target. No "buy at
  market on a feeling" — that path doesn't exist in the bot's interface.

---

## 3. Capital efficiency window (STRATEGY.md §4.6)

Idle cash compounds nothing; holding without a plan is investing, not trading. These rules force
turnover and protect mental bandwidth.

**Weekly rhythm:**

| Day | Role |
|---|---|
| Monday – Thursday | **Entry window.** New entries on A+ setups allowed. |
| Friday | **No new entries.** Manage existing (exits, target hits, stop adjustments). Review watchlist. Write the weekly journal. Mental rest. |
| Saturday – Sunday | **Closed.** No analysis, no bot prompts. Positions held with standing orders. |

**Capital deployment metric** (rolling 5-trading-day average of % of account in active positions):

| Deployed % | Bot action |
|---|---|
| < 30% for 5+ consecutive trading days | Recommend withdrawing idle cash to a separate sleeve. Idle cash isn't earning, isn't at risk, compounds nothing. |
| 30% – 80% | Normal operating range. |
| > 80% | **No new entries.** Capacity full; wait for a position to close. |

**Position duration:** most options should close within 2–10 trading days; shares may hold longer if
chart + thesis support it. Held > 15 trading days without hitting any exit condition → bot prompts a
review ("is the thesis still valid, or is this drift?"). Not a hard time stop — a "wake up and
re-examine" prompt against passive holding.

---

## 4. Anti-tilt cooldowns (STRATEGY.md §4.4)

| Trigger | Cooldown |
|---|---|
| Stop-out (loss closes at −50% premium or chart stop) | **24h** before bot proposes next entry |
| 2 consecutive losses | **48h** cooldown |
| 3 consecutive losses | **3 trading days cooldown** (extendable to 5 days at user discretion to span the weekend for emotional reset). Manual review required before resuming; log what went wrong. Sizing resets to tier base on resumption (`Trading Strategy.MD §2.3`). |
| Single +100% winner | **24h** cooldown (overconfidence is as expensive as despair) |
| Daily loss limit hit (−3% account) | Flat for the remainder of the day |
| Account drawdown ≥ −5% from rolling 30-day high | Earnings option plays explicitly blocked |
| 3 consecutive **Tier 1C** losses | Tier 1C suspended **90 days** + mandatory framework review |

**If in any cooldown state, the answer is always SKIP regardless of setup quality.**

---

## 5. Red lines — the "I will not" list (STRATEGY.md §9)

Non-negotiable. The bot refuses to execute any of these.

1. No naked short options. Defined risk only.
2. No averaging down on losing options (theta works against you — different from shares).
3. No new entries within 48h of FOMC / CPI / PPI.
4. No earnings exposure as shares (must be options if you want exposure through a print).
5. No earnings exposure during account drawdown ≥ −5%.
6. No trades in names outside Tier 1 (the watchlist).
7. **Pre-market entries allowed**, but **NO buying pre-market runners that have moved ≥ 20% from prior close.** Chasing parabolic pre-market moves is a known retail failure mode -- the bot blocks any proposed entry whose ticker has already gone up that much before regular session open. Exits remain allowed in any session.
8. No new entries after HK 23:00 (US 11:00 EDT). Management-only window allows exits/stop
   adjustments, not new positions. After HK 01:00 only standing orders run.
9. Max 3 concurrent positions in the same sector (concentration cap).
10. **Max 33% of account in any single underlying.** (Was 5% in earlier draft; raised to 33% by user 2026-06-03 to accommodate small real-account sizing where 5% wouldn't fit a single share of most names. With a meaningfully-sized account this is still a sane concentration cap.)
11. Daily loss limit: −3% of account → flat for the day, no further entries.
12. **Real-money execution enabled from the start; the mandatory paper-only period is dropped.**
    A live Futu fill placed via the CLI is the bar for "it works" — real validation, not a paper
    cycle. Real trading still requires every broker safety gate below (manual OpenD GUI unlock +
    `TRADINGBOT_ALLOW_REAL=1` + `--real`); `TrdEnv.SIMULATE` remains the default. (History: 4 weeks
    paper-only in earliest draft → 2 weeks 2026-06-03 → **dropped entirely 2026-06-07 by user**, who
    wants real-money execution as the validation. Accepted caveat: a ~$77 account proves the
    plumbing, not the strategy — size up before drawing performance conclusions, and defer
    real-money *day* trades until the account is >$5K on fee economics.)
13. No new entries on Fridays. Entry window is Mon–Thurs only.
14. No passive holding. Every open position has a defined exit plan. A position without one is a
    buy, not a trade — reject.

**Operational red lines (broker safety):**
- **Never call `unlock_trade` via the Futu SDK.** Trading unlock is done manually in the OpenD GUI.
- Default `TrdEnv.SIMULATE` (paper) everywhere. The real-trading toggle is a single explicit config
  flag, off by default.

---

## 6. The pre-trade gate checklist (10 gates, tiered)

Every gate runs (the engine evaluates all 10); how a failure is resolved depends on the gate's
**tier** — see §6.1. (Detail for each gate lives in the topic file noted.)

| # | Gate | Pass condition | Tier | Detail in |
|---|---|---|---|---|
| 1 | **Universe + Tier** | Ticker on `watchlist.yaml`; tier read; 1C override fields present; not under 1C 90-day suspension | override | `watchlist.MD` |
| 2 | **Cooldown / account state** | No active cooldown (§4); not over daily loss limit; not in blocking drawdown | **absolute** | this file §4 |
| 3 | **Day-of-week + time** | Mon–Thurs; before HK 23:00; not pre-market | soft | this file §2–3 |
| 4 | **Macro / event filter** | No FOMC/CPI/PPI within 48h; earnings handled per rule | override | `Macro Indicators.MD` |
| 5 | **A+ setup (EMS)** | S/R interaction OR valid flag/wedge + flagpole; volume confirms; chart stop identified | soft | `Technical Analysis.MD` |
| 6 | **Mode check (Bot 1)** | Mode ≠ DEFENSIVE_FLAT; mode dictates instrument shape | override | `Macro Indicators.MD` |
| 7 | **Sizing (tier-scaled)** | Position fits the tier's max-loss ceiling | **absolute** | `Trading Strategy.MD` |
| 8 | **Shares vs options** | Instrument chosen per mode + IV + account size | soft | `Trading Strategy.MD` |
| 9 | **Single-underlying concentration** | New + existing exposure in one name ≤ 33% of equity | override | this file §5 #10 |
| 10 | **Sector / theme concentration** | Sector exposure within cap (cf. §5 #9: max 3 concurrent positions/sector) | override | this file §5 #9 |

### 6.1 Tiered enforcement (user decision 2026-06-07)

Not all gates carry equal weight. A failure is resolved by tier:

- **Absolute (G2, G7)** — never proceed. The anti-tilt loss limits and the per-trade max-loss cap
  are the hard floor. **Not overridable** — these are the bot-as-override against the §0 failure
  modes, full stop.
- **Override-only (G1, G4, G6, G9, G10)** — risk / structural red lines. A failure blocks **unless**
  the operator passes an explicit, **logged `--override "<reason>"`**. A high score can **never**
  auto-carry these — breaking a concentration cap or a macro blackout must be a conscious, recorded
  act, not an algorithmic one.
- **Soft (G3, G5, G8)** — quality / timing preferences. A failure is **waived if the screener
  composite score ≥ threshold** (grade-B cutoff, `Trading Strategy.MD §7`); otherwise the trade is
  **WAIT**. This is where the quant/qual score legitimately carries a borderline setup.

**Decision rule:** ALLOW iff (all absolute pass) AND (no override-only fails, unless `--override`)
AND (any soft fails are carried by composite ≥ threshold). Enforced in code at the execution
boundary — `broker-place` refuses to place otherwise (`policy/engine.py::tiered_decision`).

When the user pushes back on an **absolute** or **override** gate ("but it looks like it's going to
break out") — restate the red line and stop. The score does not move those. Soft gates are the only
place "it looks good" is allowed to matter, and only via the screener score.

---

## 7. EMS discipline rules (STRATEGY.md §3.10)

- Do not adjust stops mid-trade unless the **chart structure** changes (e.g., a new higher low forms
  in a long).
- Limit number of trades — quality over quantity, **A+ only**.
- Control emotions — the bot is the override; if the bot says SKIP, you SKIP.

---

## 8. Structured daily routine — HK time (STRATEGY.md §3.9)

| When (HK) | Activity |
|---|---|
| 06:00–09:00 | Bot 2 overnight screener done. Read the day's plan: graded names, suggested entries, management flags. |
| 09:00–11:00 | Finalize watchlist, place limit entries, set GTC stops + targets. Pre-market gap exits if applicable. |
| 11:00–21:00 | FDT day job. No active trading. |
| 21:30–23:00 | **Preferred entry window.** Watch the open; act on fresh A+ setups; take fired targets. |
| 23:00–01:00 | **Management-only.** No new entries. Take profit targets; adjust stops only if structure changes. |
| 01:00–04:00 | **Passive.** Asleep. Standing orders run unattended. |
| Next 06:00 | Review fills, journal each trade per §9, cycle repeats. |

---

## 9. Journaling — mandatory (STRATEGY.md §10)

Every trade logged at entry and exit.

**At entry:**
- Date, time (HK), ticker, instrument (shares / option contract), size, capital at risk
- Entry trigger: which screener subsections cleared, what grade, what's missing
- Chart annotation (screenshot of daily chart with S/R + flag marked)
- Thesis in 1–2 sentences
- Stop level (chart-based) and reasoning
- Profit targets (tier-scaled)
- Bot 1 mode at the time
- Watchlist tier

**At exit:**
- Date, time, exit price, P&L (% of premium / % of capital)
- Which exit rule fired (the exit waterfall step)
- Emotional state on exit (honest 1–10 calm score)
- What was right / wrong about the trade
- Whether any rule was broken, and which one

Journals are reviewed **weekly** (Friday). Patterns of rule-breaking get flagged for STRATEGY.md
revisions. The weekly review must specifically address **Tier 1C performance** — profitable, or the
historical failure mode re-emerging?

---

## 10. What we explicitly do NOT do (STRATEGY.md §11)

1. No options pricing engine (FDM, Crank-Nicolson, Leisen-Reimer). Broker pricing suffices.
2. No Monte Carlo for American option pricing. Wrong tool.
3. No Webull HK as broker (no public dev docs at decision time). Futu OpenAPI is the rail.
4. No LEAPs sized off the 20-day Markov signal (horizon mismatch).
5. No autonomous live trading. Bot proposes, user approves, broker executes. Permanent
   human-in-the-loop.
6. No Tier 3 / speculative names through the bot, ever. Those go in a separate "Vegas account" the
   bot does not see.
7. No mid-session intraday overrides. Plans are made HK morning; orders rest during the US session.
8. No reading sentiment / chatter beyond the user's KOL judgment. The bot doesn't score X/Discord
   noise — that's the user's qualitative input.
