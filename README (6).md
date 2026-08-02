# Olist Brazilian E-Commerce — Operations & Commercial Analysis

Two analytical projects built on the Olist marketplace dataset, testing ten hypotheses across delivery performance and commercial economics.


**Toolchain:** SQL (SQLite) → Python (pandas, matplotlib, scipy) → Tableau · Excel · Power BI

**Headline findings**
- A 91.79% on-time delivery rate is achieved through a **13-day promise buffer**, not delivery speed
- Customer satisfaction collapses **26% within three days** of a missed promise and floors by day seven
- Below **R$47.49**, freight consumes over 30% of item value — a structural price floor
- Growth is entirely volume-driven: **orders +747%, AOV −11%**

---

## 1. About the dataset

[Olist Brazilian E-Commerce Public Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) — 100k orders placed on the Olist marketplace between September 2016 and October 2018.

Olist is a Brazilian marketplace that connects small merchants to major sales channels. A single customer order may contain items from multiple sellers, each shipped independently, which makes order-level and item-level grains genuinely different.

### Source tables used

| File | Rows | Role |
|---|---|---|
| `olist_orders_dataset.csv` | 99,441 | Five delivery timestamps per order |
| `olist_order_items_dataset.csv` | 112,650 | Price, freight, seller per line item |
| `olist_order_reviews_dataset.csv` | 99,224 | Review score and free-text comment |
| `olist_order_payments_dataset.csv` | 103,886 | Payment type and installment count |
| `olist_products_dataset.csv` | 32,951 | Category, weight, dimensions |
| `olist_customers_dataset.csv` | 99,441 | Customer state and city |
| `olist_sellers_dataset.csv` | 3,095 | Seller state |
| `product_category_name_translation.csv` | 71 | Portuguese → English category names |

`olist_geolocation_dataset.csv` was excluded. State-level analysis did not require lat/long, and its duplicate zip prefixes cause row fan-out on join.

### Known limitation

**The dataset contains no cost or margin data.** Profitability is not computable. Freight-to-price ratio and AOV are used as value proxies throughout, and every conclusion is framed accordingly.

---

## 2. Dataset profiling

Before any cleaning, all nine files were profiled to establish scope and identify structural risks.

### Order status distribution

| Status | Share |
|---|---|
| delivered | 97.02% |
| shipped | 1.11% |
| canceled | 0.63% |
| unavailable | 0.61% |
| invoiced / processing / created / approved | 0.63% |

Only `delivered` orders were retained. Undelivered orders have no completion timestamp and cannot contribute to delivery-time or satisfaction analysis.

### Temporal coverage

The full range spans 2016-09-04 to 2018-10-17, but the tails are unusable:

- **2016-09:** 4 orders · **2016-10:** 324 · **2016-12:** 1
- **2018-09:** 16 orders · **2018-10:** 4

Analysis window fixed at **2017-01-01 to 2018-08-31** — 20 complete months.

### Grain and fan-out risk

| Table | Rows | Unique order_id | Ratio |
|---|---|---|---|
| items | 112,650 | 98,666 | 1.14 |
| payments | 103,886 | 99,440 | 1.04 |
| reviews | 99,224 | 98,673 | 1.01 |

Three consequences:
- **Payments must be pre-aggregated** before joining, or revenue inflates by ~4%
- **Reviews must be deduplicated** — ~551 orders carry more than one review
- **Multi-item orders exist**, so `COUNTD(order_id)` and `COUNT(order_item_id)` are not interchangeable

One order has no payment record; 775 orders have no line items (canceled/unavailable, removed by the status filter anyway).

### Distribution characteristics

Both projects work with heavily right-skewed data, which determined how metrics are reported:

| Field | Mean | Median | Skew | p99 | Max |
|---|---|---|---|---|---|
| `total_days` | 12.54 | 10.21 | 3.85 | 46.0 | 209.6 |
| `price` | 119.96 | 74.90 | 8.04 | 887 | 6,735 |
| `freight_ratio` | 0.32 | 0.23 | 11.78 | 1.55 | 26.2 |
| order revenue | 137.00 | 86.50 | — | 991 | 13,440 |

**Median is reported alongside mean everywhere it matters.** An AOV of R$137 describes almost no actual order; the median of R$86.50 is the more honest figure and both appear on the dashboard.

Review scores are bimodal, not normal — 59.2% five-star, 9.8% one-star, only 8.3% three-star. The mean of 4.16 describes nobody. **% 1–2 star** is used as the satisfaction KPI instead.

---

## 3. Data cleaning

### Rules applied

| Rule | Effect |
|---|---|
| Status = `delivered` only | −2,963 orders |
| Date window Jan 2017 – Aug 2018 | −345 orders |
| Drop null `order_delivered_customer_date` | −8 orders |
| Deduplicate reviews by latest `review_creation_date` | ~551 collapsed |
| Pre-aggregate payments to order grain | prevents 4% revenue inflation |
| Flag and exclude negative journey legs | −1,384 orders |

Final analysis frames: **94,819 orders** (Project 1) and **109,880 line items / 96,211 orders** (Project 2).

### Impossible values

The three delivery legs are derived by subtracting consecutive timestamps. Some rows produced negatives, meaning the carrier scan preceded the seller approval — a real data-quality artifact, not a calculation error.

| Flag | Count |
|---|---|
| `carrier_before_approval` (leg_handling < 0) | ~1,370 |
| `neg_carrier_leg` | ~14 |
| `null_timestamp` | 14 |

Minimum `leg_handling` before cleaning was **−171 days**. These rows were flagged rather than silently dropped, and the flagged file was retained as an audit trail.

### The outlier decision — tested and reversed

Standard practice would cap `total_days` at the 99.5th percentile (~54 days) and exclude the tail. That rule was applied, then tested:

| | Excluded rows (479) | Overall (94,819) |
|---|---|---|
| Late rate | **98.5%** | 8.2% |
| Avg review score | **1.95** | 4.15 |
| % 1–2 star | **68.3%** | ~12.8% |

The excluded orders were **12× more likely to be late** and **5× more likely to receive a 1–2 star review** than average. They are not measurement errors — they are the failure cases the analysis exists to find.

**The exclusion was reversed.** Extreme outliers are retained for all satisfaction and OTDR analysis, and excluded only from stage-duration averages where a 200-day tail distorts the mean. Two frames were maintained:

- `fact_delivery_clean` (94,819) — satisfaction, OTDR, geography
- `fact_delivery_legs` (94,340) — leg duration statistics only

### Remaining caveats, disclosed

- Multi-seller orders (1.14%) use `MIN()` for `seller_state` and `payment_type` — an arbitrary pick
- 1.42% of products have no category; labelled `unknown` and retained rather than dropped
- Review comment text is Portuguese and present on only 40.2% of orders
- 643 delivered orders have no review at all

---

## 4. ETL

### Architecture

```
9 raw CSVs
    ↓  SQLite — joins, dedup, pre-aggregation, filtering
2 fact tables
    ↓  Python — feature engineering, DQ flagging, hypothesis testing
Analysis frames
    ↓  Pre-aggregated summary tables
Tableau / Power BI / Excel
```

All nine files were loaded into SQLite and two fact tables built. **Raw CSVs are never read downstream** — every visual traces back to one of the two facts.

### `fact_delivery` — one row per delivered order

Reviews deduplicated with `ROW_NUMBER()` partitioned by `order_id`; sellers aggregated to order grain before joining to avoid fan-out.

```sql
WITH rev AS (
    SELECT order_id, review_score, review_comment_message,
           ROW_NUMBER() OVER (PARTITION BY order_id
                              ORDER BY review_creation_date DESC) AS rn
    FROM reviews
),
seller_agg AS (
    SELECT i.order_id,
           COUNT(DISTINCT i.seller_id) AS seller_count,
           COUNT(*)                    AS item_count,
           MIN(s.seller_state)         AS seller_state
    FROM items i
    LEFT JOIN sellers s ON s.seller_id = i.seller_id
    GROUP BY i.order_id
)
SELECT o.order_id,
       JULIANDAY(o.order_delivered_customer_date)
         - JULIANDAY(o.order_purchase_timestamp)      AS total_days,
       JULIANDAY(o.order_delivered_customer_date)
         - JULIANDAY(o.order_estimated_delivery_date) AS days_late,
       -- three-leg decomposition
       JULIANDAY(o.order_approved_at)
         - JULIANDAY(o.order_purchase_timestamp)      AS leg_approval,
       JULIANDAY(o.order_delivered_carrier_date)
         - JULIANDAY(o.order_approved_at)             AS leg_handling,
       JULIANDAY(o.order_delivered_customer_date)
         - JULIANDAY(o.order_delivered_carrier_date)  AS leg_carrier,
       c.customer_state, sa.seller_state, sa.seller_count,
       r.review_score
FROM orders o
JOIN customers c        ON c.customer_id = o.customer_id
LEFT JOIN seller_agg sa ON sa.order_id   = o.order_id
LEFT JOIN rev r         ON r.order_id    = o.order_id AND r.rn = 1
WHERE o.order_status = 'delivered'
  AND o.order_delivered_customer_date IS NOT NULL
  AND DATE(o.order_purchase_timestamp) BETWEEN '2017-01-01' AND '2018-08-31'
```

The **three-leg decomposition** is the key engineering step. Splitting elapsed time into approval → handling → carrier is what made the root-cause hypothesis testable at all.

### `fact_sales` — one row per line item

Payments pre-aggregated to order grain before joining — the single most important step in this query.

```sql
WITH pay AS (
    SELECT order_id,
           SUM(payment_value)        AS total_payment,
           MAX(payment_installments) AS installments,
           MIN(payment_type)         AS payment_type
    FROM payments
    GROUP BY order_id
)
SELECT i.order_id, i.order_item_id, i.price, i.freight_value,
       COALESCE(t.product_category_name_english, 'unknown') AS category,
       p.product_weight_g,
       p.product_length_cm * p.product_height_cm * p.product_width_cm AS product_volume_cm3,
       pay.payment_type, pay.installments, c.customer_state
FROM items i
JOIN orders o        ON o.order_id    = i.order_id
JOIN customers c     ON c.customer_id = o.customer_id
LEFT JOIN products p ON p.product_id  = i.product_id
LEFT JOIN translation t ON t.product_category_name = p.product_category_name
LEFT JOIN pay        ON pay.order_id  = i.order_id
WHERE o.order_status = 'delivered'
  AND DATE(o.order_purchase_timestamp) BETWEEN '2017-01-01' AND '2018-08-31'
```

**Revenue is defined as `SUM(price)`, excluding freight.** `payment_value` was deliberately not used — it bundles freight and installment effects and would double-count.

### Derived fields

| Field | Definition |
|---|---|
| `is_late` | `days_late > 0` |
| `delay_bucket` | On time / 1–3 / 4–7 / 8–15 / 15+ days |
| `promised_days` | `total_days − days_late` |
| `is_negative` | `review_score <= 2` |
| `freight_ratio` | `freight_value / price` |
| `price_decile` | 10 equal-count bands |
| `inst_band` | 1 / 2–3 / 4–6 / 7–12 / 12+ |

### Serving layer

Nine pre-aggregated tables feed the BI tools so 34-point scatters don't scan 110k rows: `p1_cliff_curve` (36), `p1_state_summary` (27), `p1_lanes` (69), `p2_category_economics` (34), `p2_price_deciles` (10), plus the two facts, an order-level rollup, and a date dimension.

### Validation

Every headline number was reproduced from the exported CSVs independently of the notebook: OTDR 91.79%, median delivery 10.3d, revenue R$13,181,027.13, AOV R$137.00, freight-to-sales 16.63%.

---

## 5. Hypotheses and results

Ten hypotheses. **Seven confirmed, one rejected, one revised then confirmed, one dropped as immaterial.**

---

### Project 1 — Customer Support & Operations

#### H1 — The satisfaction cliff ✅ CONFIRMED

**Statement:** Customer satisfaction does not decline linearly with delay. It collapses at a specific threshold, beyond which further delay causes little additional damage because the customer is already lost.

**Why it matters:** A linear relationship implies "be faster generally" — unbudgetable. A threshold implies "protect this specific SLA" — a concrete operational decision with a measurable cost.

**Result:**

| Days late | Avg score | Day-on-day change |
|---|---|---|
| 0 | 4.14 | — |
| 1 | 4.00 | −0.136 |
| 2 | 3.70 | −0.295 |
| **3** | **3.06** | **−0.648** ← steepest |
| 4 | 2.62 | −0.432 |
| 7 | 1.83 | −0.298 |
| 8–25 | ~1.6–1.7 | flat within noise |

Score falls **26.1% in three days**, floors at ~1.7 by day seven, and does not decline further. By delay bucket, the share of 1–2 star reviews runs **9% → 19% → 60% → 77% → 76%**.

Of 7,788 late orders, **2,642 (33.9%) sit in the recoverable 1–3 day window**.

**Implication:** Escalate at day 2. Past day 7, additional recovery effort buys nothing — the review is already lost.

---

#### H2 — The padded promise ✅ CONFIRMED

**Statement:** The 91.79% on-time delivery rate is achieved by setting a conservative delivery estimate, not by delivering quickly.

**Why it matters:** It reframes the headline KPI as a vanity metric. A company can hit its OTDR target indefinitely while customer experience degrades.

**Result:** Median promised delivery **23.2 days** against median actual **10.3 days** — a **13-day buffer**. The gap persists across all 20 months and never narrows.

More pointedly: from March 2018 onward actual delivery improved to ~7–9 days while the promise stayed at 21–27 days. **The buffer widened as operations improved.**

**Implication:** Replace OTDR with a promise-adjusted metric, or the KPI will keep passing while satisfaction falls.

---

#### H3 — Seller handling drives delay ❌ REJECTED

**Statement:** Delivery-time variance is owned by the seller handling stage, so the fix is a seller SLA.

**Why it matters:** Determines where operational investment goes — supplier management versus carrier and lane management. Getting it wrong misdirects the entire remediation budget.

**Result:** The hypothesis is wrong.

| Stage | Mean days | Share | r with total | r with review score |
|---|---|---|---|---|
| Approval | 0.40 | 3% | 0.112 | −0.020 |
| Seller handling | 2.79 | 23% | 0.407 | −0.154 |
| **Carrier transit** | **9.08** | **74%** | **0.915** | **−0.302** |

Carrier transit dominates both duration and variance (std dev 7.39 vs 3.21 for handling).

**Implication:** The lever is carrier contracts and lane management, not seller SLAs. This rejection is more valuable than a confirmation — it redirected the recommendation entirely.

---

#### H4 — Distance from São Paulo drives failure ✅ CONFIRMED

**Statement:** Late delivery concentrates in states far from São Paulo. Because 70.7% of sellers are SP-based, geography and supply concentration drive failure, not carrier quality.

**Why it matters:** Distinguishes a structural network problem from a vendor performance problem. Only the first justifies investment in regional fulfilment.

**Result, by rate:**

| State | Late rate | Orders |
|---|---|---|
| AL | 23.72% | 392 |
| MA | 20.00% | 700 |
| PI | 16.24% | 468 |
| SP | 5.96% | 39,809 |

**Result, by volume:**

| State | Late orders |
|---|---|
| SP | 2,373 |
| RJ | 1,660 |
| MG | 634 |
| AL | 93 |

**The two views disagree completely.** Alagoas is worst by rate but 15th by volume — fixing it helps 93 customers. São Paulo has an excellent rate but the highest absolute count.

Worst lanes: **SP→AL 26.09%** (23-day median transit), SP→MA 21.68%, SP→PI 18.46%, **SP→RJ 15.72% across 8,017 orders**.

**Implication:** Prioritise SP→RJ, SP→BA and SP→CE by volume-weighted impact. Rate ranking alone would send an ops team to the wrong states.

---

#### H5 — Damage concentrates in the tail ✅ CONFIRMED

**Statement:** Customer damage is not evenly distributed. A small tail of extremely slow deliveries produces a disproportionate share of all negative reviews.

**Why it matters:** Fixing the tail beats improving the average, and the two require different interventions.

**Result:** The slowest decile (D10) alone produces **34.5% of all 1–2 star reviews**. The slowest two deciles produce **45.5% of negative reviews from 20% of orders**.

Score decay by decile is gentle through D1–D8 (4.47 → 4.18), then breaks at D9 (4.01) and collapses at D10 (2.89) — the same cliff as H1, observed through a different lens and confirming it independently.

---

### Project 2 — Sales, Finance & Revenue

#### H1 — Growth is volume-driven, not value-driven ✅ CONFIRMED

**Statement:** Revenue growth comes from acquiring more orders, not from customers spending more.

**Why it matters:** Volume growth means the lever is acquisition spend. Value growth means the lever is bundling and merchandising. The two require opposite investments.

**Result (indexed, Jan 2017 = 100):**

| Metric | Aug 2018 | Change |
|---|---|---|
| Orders | 846.8 | **+747%** |
| Revenue | 750.1 | **+650%** |
| AOV | 88.6 | **−11%** |

AOV correlation with time is −0.229. Items per order barely moved (1.217 → 1.125).

Revenue grew **slower than orders** — each incremental order is worth less than the last.

November 2017 spiked to 971.9 then fell to 735.1 in December, suggesting Black Friday pulled demand forward rather than creating it.

---

#### H2 — There is a freight break-even price floor ✅ CONFIRMED

**Statement:** Freight ratio is inversely related to item price. Below a specific price point, shipping consumes so much of order value that the transaction is structurally uneconomic.

**Why it matters:** Produces a single actionable number — the minimum viable catalogue price.

**Result:**

| Decile | Price range | Median freight ratio |
|---|---|---|
| 1 | R$0.85–23.80 | **78.5%** |
| 2 | R$23.89–34.90 | **49.0%** |
| 3 | R$34.92–47.49 | **36.4%** |
| 4 | R$47.50–59.00 | 28.8% |
| 10 | R$228+ | 6.4% |

**Break-even: R$47.49.** Correlation between price and freight ratio is −0.292 at item level.

**4,000 line items (3.64%) have freight exceeding the product price**, at a median price of R$13.90. Stated honestly, these represent only 0.53% of revenue — the magnitude is in order count and handling cost, not revenue at risk.

---

#### H3 — Revenue rank ≠ value rank ⚠️ REVISED, THEN CONFIRMED

**Original statement:** The highest-revenue categories concentrate in the high-freight-burden quadrant.

**Testing rejected this.** The ten largest categories cluster near the median freight ratio (health_beauty 0.210, bed_bath_table 0.218, sports_leisure 0.218) rather than in the danger zone.

**Revised statement:** Category freight economics vary more than five-fold and this variation is statistically independent of revenue rank.

**Why it matters:** Management ranks categories by revenue and allocates investment accordingly. If freight burden carries no relationship to that ranking, the revenue league table gives no signal about category health, and the action list must come from a second axis.

**Result:** Spearman ρ between revenue rank and freight-burden rank = **−0.219, p = 0.214** — not significant at n=34 (|ρ| > ~0.34 required). Independence cannot be rejected.

Freight ratio spans **5.4×**, from watches_gifts at 0.116 to electronics at 0.627.

**The mechanism, at category level:**

| Driver | Correlation with freight ratio |
|---|---|
| **price** | **−0.727** (R² = 0.74, p < 0.0001) |
| rev_share | −0.235 |
| volume | −0.088 |
| weight | −0.052 |

**Price drives freight ratio. Weight does not.** This overturns the intuitive answer. Illustration: office_furniture at 11,000g has a healthy 24.5% ratio; electronics at 200g has 62.7%.

**Cross-validation:** Both action-list categories fall below the H2 break-even independently derived from item-level deciles. Mean freight ratio is **0.441 below R$47.49** versus **0.225 above**. Two different methods, two different units of analysis, same shortlist.

| Action | Categories |
|---|---|
| REPRICE / KILL | electronics (0.627, R$21.89), telephony (0.434, R$29.99) — 3.80% revenue exposure |
| GROW | watches_gifts (0.116), cool_stuff (0.157), perfumery (0.171) |

---

#### H4 — Installments lift AOV ✅ CONFIRMED

**Statement:** Customers spend materially more when payment is split across installments, making installment availability a revenue lever rather than a payment option.

**Why it matters:** Installments carry a financing cost, so finance teams often want to restrict them. If they demonstrably lift basket size, restricting them destroys revenue.

**Result:**

| Band | Mean AOV | Median | Lift | % orders | % revenue |
|---|---|---|---|---|---|
| 1 | R$99.94 | R$59.99 | — | 48.5% | 35.4% |
| 2–3 | R$114.05 | R$90.00 | +14% | 23.0% | 19.1% |
| 4–6 | R$157.05 | R$107.85 | +57% | 16.3% | 18.7% |
| 7–12 | R$300.08 | R$179.00 | **+200%** | 12.0% | **26.3%** |
| 12+ | R$375.67 | R$219.90 | +276% | 0.2% | 0.5% |

**Installment users are 52% of orders but 65% of revenue.** The 7–12 band alone delivers 26% of revenue from 12% of orders.

Median rises monotonically alongside mean, so the effect is not outlier-driven.

**Caveats stated:** causality plausibly runs both ways — expensive items may require installments. The 12+ band has n=179, too small to quote.

---

#### H5 — Payment types serve different demand ✅ CONFIRMED

**Statement:** Boleto users show lower AOV and a different category mix than credit card users, making payment mix a revenue-mix variable.

**Why it matters:** Boleto has no installment facility and settles slower, but serves customers without credit access — a fifth of all transactions.

**Result:** Boleto AOV **R$121.39** versus credit card **R$142.40** — a **−14.8% gap**. Boleto is 19.9% of orders but 17.6% of revenue. Mix has been stable at roughly 80/20 across the entire period.

The category skew contradicts the naive read:

| Category | Boleto share | Median price |
|---|---|---|
| computers_accessories | **28.3%** | R$82.00 |
| garden_tools | 24.4% | R$59.90 |
| furniture_decor | 21.6% | R$67.00 |
| *— overall 20% —* | | |
| watches_gifts | 17.5% | R$129.00 |
| bed_bath_table | 17.5% | R$79.90 |

Boleto over-indexes on **considered, higher-ticket purchases** — computers at R$82 median — and under-indexes on impulse categories. These are deliberate purchases made without credit, not simply cheap ones.

---

#### Dropped — multi-seller order penalty

A hypothesis that multi-seller orders carry higher late rates was dropped after profiling. Mean `seller_count` is 1.01 with p95 = 1.00; multi-seller orders are ~1% of volume — too small to support a recommendation.

---

## 6. Dashboards

### Dashboard 1 — *A Metric That Passes While Customers Leave*

**Project 1 · Delivery performance and customer experience**

**KPI strip:** 94,819 delivered orders · 10.26 median delivery days · **12.96-day promise buffer** · 91.79% OTDR · 12.83% 1–2 star reviews

The buffer card is deliberately styled as an alert. It is the number that reframes every other figure on the page: the OTDR looks healthy only because the promise is soft.

**Charts and what they show**

*Satisfaction collapses by day 3 and floors by day 7* — the day-by-day curve. A plateau at 4.2 through the promise date, near-vertical collapse over days 1–7, then a flat floor at ~1.7 with a reference line marking it. The flat tail is the counterintuitive part: past day 7 further delay costs nothing more, which is what justifies escalating early rather than triaging the worst cases.

*60% of orders 4–7 days late receive a 1–2 star review* — the same finding by bucket, colour-coded healthy / degrading / lost. The 19% → 60% jump between the 1–3 day and 4–7 day buckets is the most quotable version of the cliff.

*Alagoas is worst by rate* and *São Paulo and Rio lose the most customers* — the rate/volume pair, deliberately placed adjacent. Same data, opposite priorities. Colour on the volume chart still encodes late rate, so the tall bars are visibly low-rate: high-volume states have good rates but generate the most absolute failures.

*SP → Alagoas: 26% of orders arrive late* — lane-level detail. SP→RJ at 15.72% across 8,017 orders is the volume-weighted priority even though seven lanes have worse rates.

*Carrier transit owns 74% of delivery time — not the seller* — the rejected hypothesis, shown rather than hidden. Approval 0.40d, seller handling 2.82d, carrier 9.37d.

**Conclusion:** A 91.79% OTDR is achieved through a 13-day promise buffer, not delivery speed. When that promise is missed, satisfaction collapses within three days and floors by day seven — 2,642 orders sit in the still-recoverable window. Carrier transit owns 74% of elapsed time, so the lever is lane management, not seller SLAs.

**Recommendations:** escalate at day 2 · prioritise SP→RJ/BA/CE by volume-weighted impact · replace OTDR with a promise-adjusted metric.

---

### Dashboard 2 — *Growth Without Value: Olist's Volume Problem*

**Project 2 · Commercial performance**

**KPI strip:** R$13,181,027 revenue · 96,211 orders · 72 categories · R$137.00 AOV (median R$86.50) · 16.63% freight-to-sales

**Charts and what they show**

*Orders grew 747%. AOV fell 11%.* — three indexed lines from Jan 2017 = 100. Orders and revenue climb together; AOV runs flat along the baseline for twenty straight months. Revenue tracks below orders from mid-2017 onward and the gap widens, meaning each incremental order is worth less than the last. The Nov-2017 Black Friday spike is annotated, along with the December pullback that suggests demand was pulled forward rather than created.

*Installment users are 52% of orders but 65% of revenue* — bars for mean AOV with a median line overlaid. Both rise monotonically, which is what rules out an outlier explanation. The 7–12 band at R$300 against R$99.94 for single payment is the +200% lift.

*Boleto AOV is 15% below credit card* — the four payment types with order-share labels.

*Boleto over-indexes on considered purchases, not cheap ones* — top-10 categories by boleto share with a reference line at the overall 20%. computers_accessories at 28.3% sits highest, at an R$82 median price — the finding that contradicts the obvious interpretation.

*São Paulo alone drives 38% of revenue* — filled map, stepped colour. Tooltips carry AOV per state, which surfaces a cross-project observation: PA has an AOV of R$184 against the national R$137, but a 12.34% late rate. The highest-value customers get the worst service.

**Conclusion:** Growth is entirely volume-driven. The available lever is payment: installment users are 52% of orders but 65% of revenue, with the 7–12 band alone delivering 26% of revenue from 12% of orders.

**Recommendations:** extend installment availability into the R$100–250 band where single-payment orders still dominate · treat boleto as a distinct considered-purchase segment rather than a payment fallback.

---

### Dashboard 3 — *The R$47.49 Problem: Where Shipping Destroys Value*

**Project 2 · Freight economics**

**Charts and what they show**

*Below R$47.49, freight consumes over 30% of item value* — ten price deciles, first three red. Each bar is labelled with its median price so the break-even is readable directly off the chart. A 30% reference line marks the threshold.

*Revenue rank tells you nothing about freight health* — the quadrant scatter. 34 categories positioned by median price and freight ratio, sized by revenue, coloured by action. electronics and telephony sit isolated top-left; watches_gifts and cool_stuff bottom-right with large bubbles. Tooltips carry both ranks, so hovering watches_gifts shows revenue rank 2 against freight rank 34 — the mismatch in a single interaction.

*Price drives freight ratio (R² = 0.74)* and *Weight does NOT (r = −0.052)* — the mechanism pair, side by side on a shared y-axis. One shows a clean logarithmic decay with a fitted trend line; the other is an unpatterned cloud. Together they disprove the intuitive answer and establish the real driver in a single glance. This pair does more analytical work than any other visual in either project.

**Right rail:** the action list — electronics and telephony to reprice or delist at 3.80% combined revenue exposure; watches_gifts, cool_stuff and perfumery to grow.

**Conclusion:** Below R$47.49, freight consumes over 30% of item value. Category freight burden varies 5.4× and is statistically independent of revenue rank (ρ = −0.219, p = 0.214). The driver is price (R² = 0.74), not weight (r = −0.052) — so the fix is a price floor, not lighter packaging.

**Recommendations:** enforce a R$47.49 catalogue price floor or bundle below it · reprice or delist electronics and telephony · the low exposure makes this a low-risk action.

---

## Repository structure

```
olist_project/
├── README.md
├── KEY_FINDINGS.md
├── notebooks/
│   └── olist_01_etl_and_eda.ipynb
├── sql/
│   ├── fact_delivery.sql
│   └── fact_sales.sql
├── processed/
│   ├── fact_delivery_clean.csv
│   ├── fact_delivery_flagged.csv
│   ├── fact_delivery_legs.csv
│   └── fact_sales.csv
├── dashboard_data/
│   ├── p1_fact_delivery.csv
│   ├── p1_cliff_curve.csv
│   ├── p1_state_summary.csv
│   ├── p1_lanes.csv
│   ├── p2_fact_sales.csv
│   ├── p2_category_economics.csv
│   ├── p2_order_level.csv
│   ├── p2_price_deciles.csv
│   └── dim_date.csv
└── dashboards/
```

---

## Notes on method

Three decisions in this analysis were reversed by their own evidence, and all three are documented rather than hidden:

1. **The p99.5 outlier exclusion was applied, tested, and reversed** when the excluded rows turned out to be the worst-served customers in the dataset.
2. **H3 in Project 1 was rejected.** The predicted seller-SLA lever does not exist; the data pointed at carrier transit instead, which changed the recommendation completely.
3. **H3 in Project 2 was revised mid-analysis** when the top-revenue categories failed to appear where predicted. The revised form produced a stronger, statistically testable claim.

Where the data contradicts a stated position, the contradiction is reported.

---

**Source:** Olist Brazilian E-Commerce Public Dataset (Kaggle, CC BY-NC-SA 4.0)
**Period analysed:** January 2017 – August 2018
**Scope:** 94,819 delivered orders · 109,880 line items · 72 product categories · 27 states
