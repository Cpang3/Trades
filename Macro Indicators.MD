# Macro Indicators

> **Compartmentalized from `STRATEGY.md` v0.3 §5.1 (Bot 1 mode classifier), §3.5, §7 (earnings + macro
> event blocks), §2 (regime windows)** plus the regime-model layer in `src/markov/`.
> This file covers the **top-down filter**: the regime/macro inputs that decide *what shape of trade
> is even allowed* on a given day, before any single name is screened.
>
> Part of a 5-file set: `watchlist.MD` · `Trading Rules.MD` · `Macro Indicators.MD` · `Technical Analysis.MD` · `Trading Strategy.MD`

---

## 1. Where macro sits in the flow

Two decision layers run **sequentially**:

1. **Bot 1 — Mode classifier** (this file): regime + macro → one **mode** for the day. Sets the
   ceiling on what's allowed.
2. **Bot 2 — Name screener** (see `Trading Strategy.MD`): a specific watchlist name → ENTER / WAIT /
   SKIP, *constrained by* Bot 1's mode.

**Mode determines what shape of trade is even allowed.** If mode = `DEFENSIVE_FLAT`, no new entry
passes the screener regardless of how good the individual name looks. This is Gate 6 of the 8-gate
checklist.

---

## 2. Bot 1 — Mode classifier (STRATEGY.md §5.1)

### Inputs

| Input | Source | What it tells us |
|---|---|---|
| **Markov regime on SPY returns** | `src/markov/system.py` | Bull / Bear / Sideways state of the broad tape |
| **VIX level + 5-day trend** | yfinance `^VIX` | Volatility regime; fear/complacency |
| **10y / 2y yield curve** | yfinance `^TNX`, `^IRX` | Rate stress, curve shape, risk appetite |
| **CPI / PPI release calendar + last surprise** | release calendar | Inflation-shock risk, recent direction of surprise |
| **FOMC meeting calendar** | 8 meetings/yr, dates known in advance | Binary policy-event risk windows |

### Output — one mode label per day

| Mode | When | Implication |
|---|---|---|
| `LONG_STOCKS_OK` | Bull regime, VIX < 15, supportive macro | Shares preferred for new entries; options only when IV unusually cheap |
| `LONG_OPTIONS_PREFERRED` | Bull regime, VIX 15–25, mixed macro | Defined-risk options only; no naked share exposure |
| `HEDGED_OR_NEUTRAL` | Sideways regime, VIX 20–30, uncertain macro | Skip directional, or use paired structures |
| `DEFENSIVE_FLAT` | Bear regime, VIX > 30, macro stress | No new entries. Manage existing only. |

The mode feeds two places:
- **Gate 6** of the pre-trade checklist (hard SKIP if `DEFENSIVE_FLAT`).
- **Shares-vs-options selection** (see `Trading Strategy.MD` §shares vs options).
- **Bot 2 "Macro Regime Fit"** subsection (10% of the screener score) + a `DEFENSIVE_FLAT`
  failure-flag that caps the grade at C.

> **Status note:** Bot 1 is **designed here and coded in `src/markov/` (Markov + HMM), but not yet
> integrated** into the live gate flow. Until wired in, the mode is supplied manually (the user
> states the current mode). The VIX/yields/calendar inputs above are the spec for the integration,
> not yet a running pipeline.

---

## 3. The regime model behind Bot 1 (`src/markov/`)

The regime input is a 3-state model of SPY (or index) daily returns.

**States (MECE):** Bull = 0, Bear = 1, Sideways = 2 — classified from the rolling return window;
warm-up rows with insufficient history are dropped (not silently bucketed as Sideways).

**Two interchangeable engines:**
- **Markov chain** — MLE transition matrix from observed state sequence; stationary distribution and
  expected dwell times describe how sticky each regime is.
- **HMM (GaussianHMM, hmmlearn)** — hidden-state inference via Viterbi decoding, with a permutation
  layer (`FittedHMM`) that maps the model's internal state order onto the external Bull/Bear/Sideways
  convention (so signals don't get scrambled by random internal labeling).

**Horizon discipline (STRATEGY.md §2):**

| Element | Value |
|---|---|
| Signal lookback (Markov) | 252 trading days |
| Regime window (Markov) | 20 days |
| Trade horizon (option DTE at entry) | 30–60 DTE |
| Average hold | 2–15 trading days |

**Hard rule:** the 20-day regime signal does **not** speak to 12-month direction. **LEAPs (200+
DTE) are out of scope** until a separate longer-horizon signal layer exists. Do not size LEAPs off
the current regime signal.

---

## 4. Macro event blocks (STRATEGY.md §3.5, §7, §9)

These are **hard calendar gates** independent of the mode. They fire even in a `LONG_STOCKS_OK`
tape.

### FOMC / CPI / PPI

- **No new entries within 48h of FOMC, CPI, or PPI releases** (red line #3). Preserves EMS's intent:
  no naked exposure to macro binary events.
- Existing positions are managed normally through these windows.
- FOMC dates are known a full year ahead (8 meetings); CPI/PPI follow the monthly BLS schedule. The
  bot maintains the calendar so Gate 4 can fire automatically.

### Earnings (STRATEGY.md §7)

Earnings are a *per-name* binary event but governed by the same macro-discipline logic.

| Account state | Position type | Earnings inside DTE | Action |
|---|---|---|---|
| Normal | Shares | Yes | Close before earnings **OR** convert to defined-risk options |
| Normal | Options | Yes | Allowed **only if the trade thesis IS the earnings catalyst** |
| **Drawdown ≥ −5%** from rolling 30d high | Any | Yes | **Hard block.** No earnings trades. |
| Normal | Any | No | No earnings restriction |

The drawdown gate is the **anti-revenge guard** — the user's diagnosed failure mode is binary
earnings gambling after losses. The bot enforces no-earnings during drawdown.

**Shares may never be held through earnings.** If you hold shares and earnings approach, the bot
prompts to convert to defined-risk options or close before the print.

---

## 5. Sentiment / KOL inputs (boundary)

The bot does **not** algorithmically score X / Discord chatter (STRATEGY.md §11 #8). Sentiment is
the **user's qualitative input** layered on top of the mode. Practical sources available:
- Futu skills (news search, stock digest, comment sentiment, capital/derivatives/technical anomaly)
  for news + flow signals.
- WebFetch on free sites.
- User-pasted tweets / KOL posts (X.com is paywalled — HTTP 402 — so it can't be fetched directly).

These inform the user's discretionary read; they are **not** an automated screener factor.
