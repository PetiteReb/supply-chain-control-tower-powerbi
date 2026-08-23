# Supply Chain Control Tower : Power BI

End-to-end Power BI analysis of **180,519 order lines** (65,752 orders, 2015–2018)
from the DataCo Smart Supply Chain dataset, built to answer one question:

**Why are 57% of deliveries late — and what would it take to fix it?**

---

## The finding

On-time delivery sits at **42.7%**. The interesting part is where it *doesn't* vary.

- **Flat across markets** — every region lands between 42% and 43%
- **Flat across time** — 36 consecutive months oscillating around the same mean

A problem that uniform is not regional and not seasonal. It is **structural**.

Tracing it to `Shipping Mode` located the cause: **the premium tiers over-promise.**

| Shipping mode | Promised | Actually delivered |
|---|---|---|
| Standard Class | 4 days | 4 days ✅ |
| Second Class | 2 days | ~4 days 🔴 |
| First Class | 1 day | 2 days 🔴 |

Customers paying for speed are the ones being let down — not because shipping is slow,
but because the commitment was never achievable.

### Quantifying the fix

Rather than stopping at a recommendation, the report includes a **what-if simulator**.
Adding a single day to the promised lead time on First and Second Class:

> **On-time delivery: 42.7% → 62.0%  ( +19.3 points )**

One day of honesty on the customer promise recovers nearly twenty points of
service performance — with no change to actual logistics.

---

## A note on measurement rigour

The first version of this dashboard reported on-time delivery at **45.2%**. That number
was wrong, and finding out why is the most useful thing in this project.

The KPI was built on `Late_delivery_risk = 0`. But a **cancelled** shipment is never late,
so its flag reads 0, meaning **2,855 cancelled orders (4.3%) were being counted as
on-time deliveries**, inflating the metric by 2.5 points.

The code was correct. The *definition* was not.

The measure was rebuilt on `Delivery Status`, so it now states its own business rule
instead of depending on an opaque flag, and orders that were never delivered are excluded
from both numerator and denominator:

```dax
On-time Delivery % =
VAR Delivered =
    FILTER( Facts_Order, Facts_Order[Delivery Status] <> "Shipping canceled" )
RETURN
DIVIDE(
    COUNTROWS(
        FILTER( Delivered,
            Facts_Order[Delivery Status] IN { "Shipping on time", "Advance shipping" } )
    ),
    COUNTROWS( Delivered )
)
```

---

## What's in the report

| Page | Purpose |
|---|---|
| **Executive Overview** | Headline KPIs, on-time by market, monthly trend with average line |
| **Delivery Performance** | The shipping-mode diagnosis — promised vs actual, by tier |
| **Delivery Simulator** | What-if parameter: adjust the promised lead time, watch on-time respond |
| **Market Detail** | Drillthrough page — right-click any market to test whether the pattern holds there |
| **Ask the Data** | Natural-language Q&A with curated synonyms and suggested questions |

---

## Technical scope

**Data preparation**: Power Query profiling on the full dataset, staging query pattern,
PII columns removed at source, star schema (1 fact + 3 dimensions + marked date table).
Grain proven, not assumed: 65,752 distinct orders across 180,519 rows.

**Modelling** : Role-playing date dimension (`USERELATIONSHIP` for order vs shipping date),
`CALCULATE`-based conditional measures, iterators with scoped `VAR` blocks, what-if
parameters on a disconnected table.

**Security** : Dynamic row-level security: one role plus a mapping table, resolved with
`USERPRINCIPALNAME()`, tested with *View as* / *Test as role*.

**Reporting** :Drillthrough with filter propagation, Q&A with a curated linguistic schema,
analytics reference lines, what-if interaction.

**Deployment** : Published to a dedicated Power BI workspace, RLS members assigned,
semantic model endorsed as *Promoted*.

---

## Roadmap

- Bookmarks and button-driven navigation
- Key influencers and decomposition tree
- Date-continuous axis to unlock trend line and forecasting
- Migration of the Q&A page to Copilot (Power BI Q&A retires December 2026)

---

*Dataset: [DataCo Smart Supply Chain](https://data.mendeley.com/datasets/8gx2fvg2k6/5) — public research dataset.
Built with Power BI Desktop and the Power BI Service.*
