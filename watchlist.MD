# Watchlist & Universe Rules

> **Compartmentalized from `STRATEGY.md` v0.3 §3.6 / §3.6.1 + `watchlist.yaml` (68 names).**
> This file defines **which names the bot is allowed to screen and trade**, the tier system that
> governs sizing/gates, and the full themed watchlist. The machine-readable source of truth is
> `watchlist.yaml`; this document is the human-readable companion.
>
> Part of a 5-file set: `watchlist.MD` · `Trading Rules.MD` · `Macro Indicators.MD` · `Technical Analysis.MD` · `Trading Strategy.MD`

---

## 1. The three-tier universe (STRATEGY.md §3.6)

The original EMS framework specifies mega-cap big tech for liquidity. The user's actual edge area
also includes narrative growth names that pass numeric filters but carry higher emotional risk. The
framework accommodates both with **tiered sizing and gates** so the user can trade their edge while
the bot enforces friction calibrated to risk.

**The watchlist is a flat list except for tier classification.** Every name on it is, by virtue of
being there, a vetted candidate. The `tier` field is a **risk-profile classification** (it determines
sizing and gates), *not* a "conviction" sub-classification. The existence test on the watchlist IS
the conviction test.

### Tier 1A — Quality mega caps (auto-pass)

```
AAPL, MSFT, AMZN, GOOGL, TSLA, NVDA, META, AMD, SPY, QQQ
```

Mag 7 + AMD + the two largest index ETFs. Structural compounders, deep liquidity.
**Standard sizing (1.0% base), standard gates.**

> Note: the canonical Tier 1A list above is the STRATEGY.md §3.6 list. The live `watchlist.yaml`
> currently carries **4 of these** as actively tracked 1A names (NVDA, AMD, GOOGL, META); the rest
> remain auto-pass per the rule even if not separately enumerated in the YAML.

### Tier 1B — Narrative growth names (auto-pass §3.6 numerics)

Any name passing **all** numeric filters below, with a written thesis on the watchlist:

- Market cap ≥ **$5B**
- Average daily volume ≥ **1M shares**
- Listed on NYSE or NASDAQ main board (no OTC, no pink sheets)
- **≥ 90 days since IPO**
- Not a SPAC pre-de-SPAC
- Not pre-revenue biotech
- Has liquid options: OI > 1000 on the specific contract, bid-ask < 5% of mid

Tradeable at **half sizing (0.5% base)**, **grade-A only**, with stricter gates and tighter exit targets.

### Tier 1C — Override trades (fails §3.6 numerics)

Names that fail one or more numeric filters above (most commonly sub-$5B mcap) may still be traded
**only via deliberate override**, requiring:

- A long-form written override thesis on the watchlist
- Acknowledgment of which numeric filter is being bypassed
- Comparison to the user's historical failure-mode tickers (**AIP / SRPT / WULF / BMMR**) and an
  explicit articulation of what makes this name different

Tradeable at **quarter sizing (0.25% base)**, **grade-A only**, tightest exit targets, plus the
override-tracking discipline in §2 below.

### Hard exclusions — no override allowed (never trade through the bot, any tier)

- OTC / pink sheets
- IPO < 90 days
- SPACs pre-de-SPAC
- Pre-revenue biotech
- Sub-1M ADV
- No listed options OR option OI < 1000 / bid-ask > 5% of mid (for option trades)

If the user wants to trade names in the hard-exclusion set, that happens in a separate **"Vegas
account"** the bot does not see.

---

## 2. Tier 1C override discipline (STRATEGY.md §3.6.1)

Tier 1C is the deliberate carve-out for high-conviction narrative trades on sub-$5B names. Because
the user has explicitly diagnosed this category (small-cap, narrative-driven) as their historical
failure mode, override trades carry additional structural friction.

**Per-name override requirements (in `watchlist.yaml`):**
- `tier: 1C`
- `override_reason`: which numeric filter is being bypassed and why
- `failure_mode_check`: paragraph comparing this name to AIP/SRPT/WULF/BMMR and articulating what
  makes it different (real catalyst? real fundamentals? real moat? or just narrative hype?)

**Per-trade friction:**
- Sizing: 0.25% base / 0.5% cap (quarter of Tier 1A)
- Screener gate: grade **A only**, no B-grade entries
- Profit targets: tighter (see `Trading Strategy.MD` §exit waterfall)
- Stop: chart-based **AND** −25% premium, whichever fires first

**Discipline limits:**
- **3 consecutive losing Tier 1C trades → Tier 1C suspended for 90 days**, mandatory framework review
  before resumption
- Weekly journal review must specifically address Tier 1C performance: is this category profitable,
  or is it the historical failure mode re-emerging?

---

## 3. Watchlist mechanism (STRATEGY.md §4.5)

The user maintains the watchlist as `watchlist.yaml` in the repo. **The bot only screens names that
appear in this file.** Adding or removing a name is a deliberate written act — one name per commit
ideally.

**Schema (Tier 1A / 1B):**

```yaml
- ticker: NVDA
  tier: 1A                          # 1A | 1B | 1C
  theme: semi_ai
  thesis: "AI capex cycle, KOL bullish, secular leader"
  added: 2026-05-14
  earnings_date: 2026-08-21         # bot can auto-update
  notes: ""
  patterns_to_watch: [flag_breakout, support_hold]
```

**Tier 1C requires additional fields:**

```yaml
- ticker: VELO
  tier: 1C
  theme: hardware
  thesis: "[normal thesis]"
  override_reason: "Sub-$5B mcap (~$750M) failing §3.6 numeric filter; overridden on turnaround thesis"
  failure_mode_check: "Pattern similarity to AIP/SRPT/WULF/BMMR. Differentiated by: [specific differentiators]"
  added: 2026-05-14
```

> **`[DRAFT — review]` marker:** any `thesis` / `override_reason` / `failure_mode_check` field tagged
> `[DRAFT — review]` was authored by Claude from public information and **needs user review/edit
> before live trading.** User-supplied content is unmarked.

---

## 4. The watchlist — 68 names by theme

Counts (as of 2026-05-14): **Tier 1A: 4 · Tier 1B: 37 · Tier 1C: 27 · Total: 68.**
Themes: semi_ai, photonics, ai_software, crypto, space, defense, fintech, energy, hardware,
healthcare, industrial.

### Semi / AI hardware (13)

| Ticker | Tier | Thesis (short) |
|---|---|---|
| **NVDA** | 1A | AI compute leader. Data-center capex, structural demand. |
| **AMD** | 1A | AI / data-center semi co-leader. MI300 / MI400 cycle. |
| TSM | 1B | Foundry monopoly for leading-edge AI silicon (~$2.1T). |
| MU | 1B | HBM memory for AI accelerators. Pricing cycle (~$1T). |
| INTC | 1B | Turnaround: foundry buildout, CPU recovery, CHIPS support (~$621B). |
| ARM | 1B | CPU IP licensing — smartphones + AI accelerators (~$342B, 11% short). |
| SNDK | 1B | NAND flash post-WDC spinoff. Memory pricing cycle (~$235B, 11% short). |
| MRVL | 1B | Custom AI silicon (Trainium ASICs) + networking (~$182B). |
| QCOM | 1B | Mobile semi pivoting to on-device AI; sold off (~$262B, 26M ADV). |
| AEHR | 1C | SiC burn-in test equipment. Sub-$5B (~$3.5B). |
| AOSL | 1C | Power-management chips. Sub-$5B (~$1.4B). |
| SKYT | 1C | US pure-play foundry, defense/govt exposure. Sub-$5B (~$1.8B). |
| AIP | 1C | Arteris — SoC interconnect IP (~$1.75B). **User override** (prior winner; also on historical-loss list). |

### Photonics / optics (4) — *group override, see POET entry in YAML for shared thesis*

| Ticker | Tier | Thesis (short) |
|---|---|---|
| POET | 1C | Photonic interconnect for AI data-center optical I/O (~$2.3B). |
| LASR | 1C | nLIGHT — industrial + defense lasers; DARPA pipeline (~$4.6B). |
| LPTH | 1C | LightPath — optical components / IR lenses; defense (~$1B). |
| OUST | 1C | Ouster — consolidated US lidar (Velodyne merger); AV/robotics (~$2.7B). |

### AI software / cloud / data (13)

| Ticker | Tier | Thesis (short) |
|---|---|---|
| **GOOGL** | 1A | Alphabet — search + Gemini + Google Cloud AI. |
| **META** | 1A | Meta — social + Llama AI + Reality Labs. |
| ORCL | 1B | Oracle Cloud Infrastructure — AI training capacity; NVDA partnership (~$555B). |
| NOW | 1B | ServiceNow — enterprise workflow + AI agents (~$103B). |
| CRWV | 1B | CoreWeave — GPU-as-a-service hyperscaler (~$58B, 20% short, recent IPO). |
| NBIS | 1B | Nebius — AI-native cloud (ex-Yandex) (~$53B, 20% short). |
| APP | 1B | AppLovin — mobile ad tech + AI monetization (~$173B). |
| SHOP | 1B | Shopify — e-commerce + AI merchant tooling (~$136B). |
| AKAM | 1B | Akamai — CDN + edge + cybersecurity (~$22B, 13% short). |
| RBLX | 1B | Roblox — gaming / metaverse / creator economy (~$33B). |
| ZETA | 1C | Zeta Global — AI marketing data platform. Borderline (~$4.9B). |
| INFQ | 1C | Infleqtion — quantum / neutral-atom; recent IPO (~$3.5B, 98 days). |
| FSLY | 1C | Fastly — edge compute / CDN turnaround (~$2.75B). |
| PDYN | 1C | Palladyne AI — physical-AI / robotics autonomy (~$330M). |

### Crypto / digital assets (9)

| Ticker | Tier | Thesis (short) |
|---|---|---|
| COIN | 1B | Coinbase — largest US crypto exchange; USDC custody (~$47B). |
| IREN | 1B | IREN — BTC miner pivoting to AI/HPC leasing (~$21B, 17% short). |
| WULF | 1B | TeraWulf — BTC miner + HPC/AI hosting (~$12.5B, 25% short). **Historical-loss flag; user-kept.** |
| GLXY | 1B | Galaxy Digital — diversified crypto financial services (~$11.6B). |
| CIFR | 1B | Cipher Mining — BTC miner (~$9.4B, 17% short). |
| CRCL | 1B | Circle — USDC issuer; GENIUS Act narrative (~$28B). **Compliance flag — user-accepted.** |
| PURR | 1C | Hyperliquid Strategies — Hyperliquid DEX proxy (~$1.1B, 174 days). |
| DXYZ | 1C | Destiny Tech100 — CEF for private tech/crypto (SpaceX, OpenAI) (~$1.8B). |
| VNET | 1C | VNET — China data-center w/ crypto/AI compute (~$2.8B). |

### Space / satellite (8)

| Ticker | Tier | Thesis (short) |
|---|---|---|
| RKLB | 1B | Rocket Lab — launch + space systems; Neutron catalyst (~$83B, 31M ADV). |
| ASTS | 1B | AST SpaceMobile — sat-to-cell; AT&T/Verizon (~$46B, 18% short). |
| SATS | 1B | EchoStar — satellite + spectrum; Boost Mobile (~$36B, 36% short). |
| PL | 1B | Planet Labs — Earth observation; defense + commercial (~$17B). |
| FLY | 1B | Firefly Aerospace — small-launch + Blue Ghost; recent IPO (~$9.4B, 292 days). |
| LUNR | 1B | Intuitive Machines — lunar landers, NASA CLPS (~$5.6B, 21% short). |
| RDW | 1C | Redwire — in-space manufacturing; defense/civil. Borderline (~$4.4B). |
| SPIR | 1C | Spire Global — space-as-a-service data subscriptions (~$920M). |

### Defense / drones (6)

| Ticker | Tier | Thesis (short) |
|---|---|---|
| PLTR | 1B | Palantir — defense/commercial AI (Gotham, Foundry, AIP); sold off (~$327B). |
| LMT | 1B | Lockheed Martin — defense prime (F-35, missiles, space) (~$135B). |
| KTOS | 1B | Kratos — UAVs, drones, satellite, hypersonics (~$11B). |
| AMTM | 1B | Amentum — engineering/mission support for DoD. Borderline (~$5.6B). |
| RCAT | 1C | Red Cat — military drones; Army SRR contract (~$1.5B). |
| UMAC | 1C | Unusual Machines — NDAA-compliant drone parts (~$830M). |

### Fintech (4)

| Ticker | Tier | Thesis (short) |
|---|---|---|
| HOOD | 1B | Robinhood — retail brokerage + crypto + tokenization (~$67B). |
| NU | 1B | Nu Holdings — LatAm digital bank (~$63B). |
| SOFI | 1B | SoFi — digital bank + lending + tech platform (~$21B, 13% short). |
| RKT | 1B | Rocket Companies — mortgage; rate-cut sensitivity (~$40B). |

### Energy / climate (5)

| Ticker | Tier | Thesis (short) |
|---|---|---|
| FSLR | 1B | First Solar — thin-film solar; IRA/tariff beneficiary (~$29B). |
| RUN | 1C | Sunrun — residential solar leasing; turnaround (~$3.5B). |
| AMRC | 1C | Ameresco — energy efficiency / federal contracts (~$1.9B). |
| AMPX | 1C | Amprius — silicon-anode batteries; defense + aviation (~$2.5B). |
| UAMY | 1C | US Antimony — critical-mineral production; reshoring (~$1.3B). |

### Hardware / computing (3)

| Ticker | Tier | Thesis (short) |
|---|---|---|
| DELL | 1B | Dell — AI server cycle (PowerEdge); enterprise + hyperscaler (~$198B). |
| OSS | 1C | One Stop Systems — rugged edge-compute / GPU for defense (~$430M). |
| VELO | 1C | Velo3D — metal additive mfg; debt cut + earnings beat; 25.6% short (~$750M). **User override.** |

### Healthcare (1)

| Ticker | Tier | Thesis (short) |
|---|---|---|
| TMDX | 1C | TransMedics — Organ Care System; only commercial normothermic perfusion (~$2.4B, 37% short). |

### Industrial (1)

| Ticker | Tier | Thesis (short) |
|---|---|---|
| SHMD | 1C | SCHMID Group — specialty industrial machinery (semi/solar/glass/H2) (~$430M, 755 days). |

---

## 5. Special flags carried on specific names

- **AIP** — On the user's historical-loss list **and** prior-winner list. User-kept as a deliberate
  override based on positive prior track record. Trade with full awareness of the dual flag.
- **WULF** — On the historical-loss list. Tier 1B (formal §3.6.1 override not required), but the
  historical-loss flag stays for journal awareness. User-kept: "Companies I've made money before."
- **CRCL** — Compliance flag: the user works at First Digital Trust (issuer of FDUSD, a competing
  stablecoin). User has stated FDT is private with no public-company personal-trading policy and has
  accepted the position. Concern is on record; user judgment governs.
- **VELO** — Full user-authored override thesis (debt reduction + earnings beat + squeeze dynamics).
- **Borderline-mcap 1C names** (ZETA, RDW, AMTM, LUNR) — sit near the $5B threshold; may migrate
  between 1B/1C on price moves. Re-check tier on meaningful rallies/pullbacks.
- **Recent-IPO caution** (INFQ 98 days, FLY 292 days, PURR 174 days) — post-IPO volatility and
  lockup dynamics may dominate fundamentals near-term.

> Every name marked `[DRAFT — review]` in `watchlist.yaml` still needs user sign-off on its thesis
> before live trading.
