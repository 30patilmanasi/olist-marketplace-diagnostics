# Key Findings

**Olist Brazilian E-Commerce · January 2017 – August 2018**
94,819 delivered orders · 109,880 line items · 72 categories · 27 states

A condensed reference of every number produced by this analysis. Full method in [README.md](README.md).

---

## The four headlines

| # | Finding | Evidence |
|---|---|---|
| 1 | A 91.79% on-time rate is achieved through a **13-day promise buffer**, not delivery speed | Promised 23.2d vs actual 10.3d, stable across 20 months |
| 2 | Satisfaction collapses **26% within three days** of a missed promise and floors by day seven | Score 4.14 → 3.06 → 1.83, then flat |
| 3 | Below **R$47.49**, freight consumes over 30% of item value | Decile 3 ratio 36.4%, decile 4 28.8% |
| 4 | Growth is entirely volume-driven: **orders +747%, AOV −11%** | Indexed to Jan 2017 = 100 |

---

## Project 1 — Delivery Performance

### Baseline

| Metric | Value |
|---|---|
| Delivered orders | 94,819 |
| On-time delivery rate | 91.79% |
| Late orders | 7,788 (8.21%) |
| Median delivery | 10.26 days |
| Median promised | 23.2 days |
| Promise buffer | **12.96 days** |
| Avg review score | 4.15 |
| 1–2 star reviews | 12.83% |
| Orders with review text | 40.2% |

### H1 — Satisfaction cliff ✅

Steepest single-day drop at **day 3 (−0.648)**.

| Days late | Score | Change |
|---|---|---|
| 0 | 4.14 | — |
| 1 | 4.00 | −0.136 |
| 2 | 3.70 | −0.295 |
| **3** | **3.06** | **−0.648** |
| 4 | 2.62 | −0.432 |
| 5 | 2.50 | −0.127 |
| 6 | 2.13 | −0.368 |
| 7 | 1.83 | −0.298 |
| 8–25 | ~1.6–1.7 | flat within noise |

By bucket:

| Bucket | Orders | Score | % 1–2 star |
|---|---|---|---|
| On time | 87,031 | 4.29 | 9.14% |
| 1–3 days | 2,642 | 3.76 | 19.00% |
| 4–7 days | 1,812 | 2.32 | 59.71% |
| 8–15 days | 1,957 | 1.73 | 76.75% |
| 15+ days | 1,377 | 1.72 | 75.60% |

Decline day 0→3: **−26.1%**. Recoverable window (1–3 days late): **2,642 orders = 33.9% of late volume**.

> **Action:** escalate at day 2. Past day 7 the review is already lost and additional recovery effort buys nothing.

### H2 — Padded promise ✅

Median promised **23.2d** vs actual **10.3d** = **13-day buffer**, persisting across all 20 months.

From March 2018 actual delivery improved to ~7–9 days while the promise held at 21–27 days. **The buffer widened as operations improved.**

> **Action:** replace OTDR with a promise-adjusted metric.

### H3 — Root cause ❌ REJECTED

Predicted seller handling; the data says carrier.

| Stage | Mean days | Share | Std dev | r with total | r with score |
|---|---|---|---|---|---|
| Approval | 0.40 | 3% | 0.80 | 0.112 | −0.020 |
| Seller handling | 2.79 | 23% | 3.21 | 0.407 | −0.154 |
| **Carrier transit** | **9.08** | **74%** | **7.39** | **0.915** | **−0.302** |

> **Action:** the lever is carrier contracts and lane management, not seller SLAs.

### H4 — Geography ✅

Seller supply is **70.7% São Paulo-concentrated**; customer demand is 41.99% SP.

**By rate:**

| State | Late rate | Orders |
|---|---|---|
| AL | 23.72% | 392 |
| MA | 20.00% | 700 |
| PI | 16.24% | 468 |
| CE | 15.72% | 1,247 |
| SE | 15.45% | 330 |
| SP | 5.96% | 39,809 |
| RO | 2.89% | 242 |

**By volume:**

| State | Late orders | Rate |
|---|---|---|
| SP | 2,373 | 5.96% |
| RJ | 1,660 | 13.64% |
| MG | 634 | 5.69% |
| BA | 452 | 14.09% |
| RS | 382 | 7.29% |
| AL | 93 | 23.72% |

**Alagoas ranks 1st by rate, 15th by volume.**

**Worst lanes (n ≥ 150):**

| Lane | Late rate | Orders | Median days |
|---|---|---|---|
| SP → AL | 26.09% | 253 | 22.97 |
| SP → MA | 21.68% | 475 | 19.68 |
| SP → PI | 18.46% | 325 | 16.82 |
| SP → SE | 16.67% | 204 | 17.83 |
| **SP → RJ** | **15.72%** | **8,017** | **12.89** |
| SP → CE | 15.61% | 948 | 18.43 |
| SP → BA | 15.01% | 2,272 | 17.71 |

> **Action:** prioritise SP→RJ, SP→BA, SP→CE by volume-weighted impact. Rate ranking alone sends resources to the wrong states.

### H5 — Tail concentration ✅

| Decile | Median days | Score | Share of all 1–2 star |
|---|---|---|---|
| D1 | 3.03 | 4.47 | 5.4% |
| D5 | 9.43 | 4.31 | 7.2% |
| D8 | 15.77 | 4.18 | 8.7% |
| D9 | 19.91 | 4.01 | 11.0% |
| **D10** | **29.31** | **2.89** | **34.5%** |

Slowest two deciles = **45.5% of all negative reviews from 20% of orders**.

### Dropped

Multi-seller order penalty. Mean `seller_count` 1.01, p95 = 1.00 — ~1% of volume, too small to act on.

---

## Project 2 — Commercial Performance

### Baseline

| Metric | Value |
|---|---|
| Revenue (item price) | R$ 13,181,027 |
| Total freight | R$ 2,192,093 |
| Freight-to-sales | 16.63% |
| Orders | 96,211 |
| Line items | 109,880 |
| AOV mean / median | R$ 137.00 / R$ 86.50 |
| Items per order | 1.14 |
| Categories | 72 (1.42% unknown) |
| Top-20 categories | 84.1% of revenue |
| Top-14 categories | 80% of revenue |

Payment mix: credit card 76.7% · boleto 20.3% · debit 1.5% · voucher 1.5%
Revenue by state: SP 38.4% · RJ 13.3% · MG 11.8% · RS 5.5% · PR 5.0%

### H1 — Volume, not value ✅

Indexed, Jan 2017 = 100:

| Metric | Aug 2018 | Change |
|---|---|---|
| Orders | 846.8 | **+747%** |
| Revenue | 750.1 | **+650%** |
| AOV | 88.6 | **−11%** |

AOV correlation with time **−0.229**. Items/order 1.217 → 1.125.

Revenue grew slower than orders — each incremental order is worth less than the last.

Nov 2017 peaked at 971.9, fell to 735.1 in December: Black Friday pulled demand forward rather than creating it.

### H2 — Freight break-even ✅

| Decile | Price range | Median price | Median freight | Ratio | Rev share |
|---|---|---|---|---|---|
| 1 | 0.85–23.80 | R$17 | R$14.10 | **78.5%** | 1.4% |
| 2 | 23.89–34.90 | R$29 | R$14.52 | **49.0%** | 2.5% |
| 3 | 34.92–47.49 | R$40 | R$15.10 | **36.4%** | 3.3% |
| 4 | 47.50–59.00 | R$51 | R$15.14 | 28.8% | 4.4% |
| 5 | 59.10–74.90 | R$66 | R$16.18 | 24.6% | 5.4% |
| 10 | 228.09–6735 | R$349 | R$23.60 | 6.4% | 40.7% |

**Break-even: R$47.49.** Price ↔ freight ratio correlation **−0.292**.

**4,000 line items (3.64%) have freight exceeding price**, median price R$13.90 — but only 0.53% of revenue. The cost is in order count and handling, not revenue at risk.

Revenue below the break-even: 7.2% of total.

### H3 — Rank independence ⚠️ REVISED → ✅

Original hypothesis (top-revenue categories cluster in the high-freight quadrant) **not supported** — the ten largest sit near the median.

Revised and confirmed: **Spearman ρ = −0.219, p = 0.214** (n = 34; |ρ| > ~0.34 needed for significance). Revenue rank carries no reliable information about freight health.

Freight ratio spread: **5.4×** (0.116 watches_gifts → 0.627 electronics), median 0.220.

**Mechanism, category level:**

| Driver | Correlation |
|---|---|
| **price** | **−0.727** (R² = 0.74, p < 0.0001) |
| rev_share | −0.235 |
| volume | −0.088 |
| weight | −0.052 |

Price drives freight ratio; weight does not. Illustration: office_furniture at 11,000g has a 24.5% ratio; electronics at 200g has 62.7%.

**Cross-validation with H2:** both action-list categories fall below the independently derived R$47.49 floor. Mean freight ratio **0.441 below** vs **0.225 above**.

| Action | Category | Ratio | Price | Rev share |
|---|---|---|---|---|
| REPRICE/KILL | electronics | 0.627 | R$21.89 | 1.27% |
| REPRICE/KILL | telephony | 0.434 | R$29.99 | 2.53% |
| GROW | watches_gifts | 0.116 | R$128.90 | 9.51% |
| GROW | cool_stuff | 0.157 | R$129.90 | 4.98% |
| GROW | perfumery | 0.171 | R$84.99 | 3.15% |

Combined kill-list exposure: **3.80% of revenue** — low-risk to act on.

**Top categories by revenue:**

| Category | Rev share | Median price | Freight ratio | Median weight |
|---|---|---|---|---|
| health_beauty | 10.05% | R$79.90 | 0.210 | 400g |
| watches_gifts | 9.51% | R$128.90 | 0.116 | 346g |
| bed_bath_table | 8.36% | R$79.57 | 0.218 | 1,275g |
| sports_leisure | 7.78% | R$77.90 | 0.218 | 700g |
| computers_accessories | 7.26% | R$81.99 | 0.208 | 300g |
| furniture_decor | 5.77% | R$65.49 | 0.270 | 1,300g |
| housewares | 5.02% | R$59.70 | 0.290 | 1,200g |

### H4 — Installment lift ✅

| Band | Mean AOV | Median | Lift | Orders | % orders | % revenue |
|---|---|---|---|---|---|---|
| 1 | R$99.94 | R$59.99 | — | 46,709 | 48.5% | 35.4% |
| 2–3 | R$114.05 | R$90.00 | +14% | 22,100 | 23.0% | 19.1% |
| 4–6 | R$157.05 | R$107.85 | +57% | 15,686 | 16.3% | 18.7% |
| 7–12 | R$300.08 | R$179.00 | **+200%** | 11,535 | 12.0% | **26.3%** |
| 12+ | R$375.67 | R$219.90 | +276% | 179 | 0.2% | 0.5% |

**Installment users: 52% of orders → 65% of revenue.** The 7–12 band alone delivers 26% of revenue from 12% of orders.

Correlation installments ↔ order revenue: **+0.314**. Median rises monotonically, so the effect is not outlier-driven.

**Caveats:** causality plausibly runs both ways — expensive items may require installments. The 12+ band (n=179) is too small to quote.

### H5 — Payment segments ✅

| Type | Mean AOV | Median | % orders | % revenue |
|---|---|---|---|---|
| credit_card | R$142.40 | R$89.90 | 77.0% | 80.1% |
| boleto | R$121.39 | R$74.90 | 19.9% | 17.6% |
| debit_card | R$119.88 | R$70.45 | 1.5% | 1.3% |
| voucher | R$86.05 | R$54.00 | 1.6% | 1.0% |

Boleto AOV gap: **−14.8%**. Mix stable at ~80/20 across all 20 months.

**Boleto share by category:**

| Category | Boleto % | Median price |
|---|---|---|
| computers_accessories | **28.30%** | R$82.00 |
| garden_tools | 24.37% | R$59.90 |
| furniture_decor | 21.59% | R$67.00 |
| sports_leisure | 21.20% | R$78.92 |
| auto | 20.58% | R$87.00 |
| *— overall 19.9% —* | | |
| housewares | 19.71% | R$59.90 |
| health_beauty | 19.69% | R$79.99 |
| bed_bath_table | 17.46% | R$79.90 |
| watches_gifts | 17.54% | R$129.00 |

Boleto over-indexes on **considered, higher-ticket purchases**, not cheap ones — computers at an R$82 median is the highest-boleto category. These are deliberate purchases made without credit.

---

## Cross-project observation

Northern states show **higher AOV and worse service simultaneously**. Pará: AOV R$184.06 against the national R$137, with a 12.34% late rate and 20.92-day median transit on the SP→PA lane.

The highest-value customers receive the worst delivery experience.

---

## Data quality decisions

### Applied

- `delivered` status only (97.02% of orders)
- Window Jan 2017 – Aug 2018 (2016 and Sep/Oct 2018 are partial)
- 8 delivered orders with null delivery timestamp dropped
- Reviews deduplicated by latest `review_creation_date` (~551 orders had multiples)
- Payments pre-aggregated to order grain before joining (1.04 fan-out ratio)
- Negative journey legs flagged and excluded (min `leg_handling` was −171 days)

### The outlier exclusion — tested and reversed

A p99.5 cap on `total_days` (~54 days) was applied, then tested against the excluded rows:

| | Excluded (479) | Overall (94,819) |
|---|---|---|
| Late rate | **98.5%** | 8.2% |
| Avg score | **1.95** | 4.15 |
| % 1–2 star | **68.3%** | ~12.8% |

These are not measurement errors — they are the failure cases the analysis exists to find. **The exclusion was reversed.** Extremes are retained for satisfaction and OTDR analysis and excluded only from stage-duration averages, where a 200-day tail distorts the mean.

### Reporting conventions

Both projects work with heavily right-skewed data:

| Field | Mean | Median | Skew |
|---|---|---|---|
| `total_days` | 12.54 | 10.21 | 3.85 |
| `price` | 119.96 | 74.90 | 8.04 |
| `freight_ratio` | 0.32 | 0.23 | 11.78 |
| order revenue | 137.00 | 86.50 | — |

Median is reported alongside mean wherever it matters.

Review scores are bimodal — 59.2% five-star, 9.8% one-star, 8.3% three-star. The mean of 4.16 describes almost nobody, so **% 1–2 star** is used as the satisfaction KPI.

### Disclosed limitations

- **No cost or margin data exists in Olist.** Profitability is not computable; freight ratio and AOV serve as value proxies
- Multi-seller orders (1.14%) use `MIN()` for `seller_state` and `payment_type` — an arbitrary pick
- 1.42% of products have no category; labelled `unknown` and retained
- Review text is Portuguese, present on 40.2% of orders
- 643 delivered orders carry no review

---

## Recommendations

### Operations

1. **Escalate at day 2**, not on the worst cases. 2,642 orders sit in the recoverable window; past day 7 the review is already lost.
2. **Prioritise by volume, not rate.** SP→RJ (8,017 orders, 15.72% late) outweighs SP→AL (253 orders, 26.09% late) by an order of magnitude.
3. **Replace OTDR** with a promise-adjusted metric. The current KPI passes on a 13-day buffer.
4. **Target the carrier leg.** It owns 74% of elapsed time and correlates 0.915 with total duration; seller SLAs will not move the number.

### Commercial

5. **Enforce a R$47.49 catalogue price floor**, or bundle below it. Two independent methods converge on this threshold.
6. **Reprice or delist electronics and telephony.** Combined exposure is 3.80% of revenue — low-risk.
7. **Extend installments into the R$100–250 band**, where single-payment orders still dominate. The 7–12 band shows a +200% AOV lift.
8. **Treat boleto as a considered-purchase segment**, not a payment fallback. It over-indexes on computers and garden tools, not cheap goods.

---

## Method notes

Three decisions in this analysis were reversed by their own evidence:

1. **The p99.5 outlier exclusion** was applied, tested, and reversed when the excluded rows proved to be the worst-served customers in the dataset.
2. **Project 1 H3 was rejected.** The predicted seller-SLA lever does not exist; carrier transit owns the delay, which changed the recommendation entirely.
3. **Project 2 H3 was revised mid-analysis** when top-revenue categories failed to appear where predicted. The revised form produced a stronger, statistically testable claim.

Where the data contradicts a stated position, the contradiction is reported.
