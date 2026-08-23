# Supply Chain Control Tower, Power BI

End-to-end Power BI analysis of **180,519 order lines** (65,752 orders, 2015-2018)
from the DataCo Smart Supply Chain dataset, built to answer one question:

**Why are 57% of deliveries late, and what would it take to fix it?**

---

## The finding

On-time delivery sits at **42.7%**. The interesting part is where it *doesn't* vary.

* **Flat across markets**: every region lands between 42% and 43%
* **Flat across time**: 36 consecutive months oscillating around the same mean

A problem that uniform is not regional and not seasonal. It is **structural**.

Tracing it to `Shipping Mode` located the cause: **the premium tiers over-promise.**

| Shipping mode | Orders | Promised | Actually delivered | On-time |
|---|---:|---:|---:|---:|
| Standard Class | 39,324 | 4.00 days | 4.00 days | **60.2%** |
| Same Day | 3,571 | 0.00 days | 0.48 days | **52.1%** |
| Second Class | 12,778 | 2.00 days | 3.99 days | **20.2%** |
| First Class | 10,079 | 1.00 day | 2.00 days | **0%** |

Read the last row again. **First Class, the premium tier, has a 0% on-time rate.**
Every one of its 10,079 orders takes two days against a one-day commitment. Delivery
performance is not erratic here, it is consistently and precisely one day short.

Customers paying for speed are the ones being let down. Not because shipping is slow,
but because the commitment was never achievable.

### Quantifying the fix

Rather than stopping at a recommendation, the report includes a **what-if simulator**.
Adding a single day to the promised lead time on First and Second Class:

> **On-time delivery: 42.7% to 62.0% (+19.3 points)**

One day of honesty on the customer promise recovers nearly twenty points of service
performance, with no change to actual logistics.

---

## A note on measurement rigour

The first version of this dashboard reported on-time delivery at **45.2%**. That number
was wrong, and finding out why is the most useful thing in this project.

The KPI was built on `Late_delivery_risk = 0`. But a **cancelled** shipment is never late,
so its flag reads 0. That meant **2,855 cancelled orders (4.3%) were being counted as
on-time deliveries**, inflating the metric by 2.5 points and, in First Class, manufacturing
an entirely fictional 4.7% success rate out of nothing but cancellations.

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
    ) + 0,
    COUNTROWS( Delivered )
)
```

---

## What's in the report

| Page | Purpose |
|---|---|
| **Executive Overview** | Headline KPIs, on-time by market, monthly trend with average line |
| **Delivery Performance** | The shipping-mode diagnosis: promised vs actual, by tier |
| **Delivery Simulator** | What-if parameter: adjust the promised lead time, watch on-time respond |
| **Market Detail** | Drillthrough page: right-click any market to test whether the pattern holds there |
| **Ask the Data** | Natural-language Q&A with curated synonyms and suggested questions |

---

## Technical scope

**Data preparation**: Power Query profiling on the full dataset, staging query pattern,
PII columns removed at source, star schema (1 fact, 3 dimensions, marked date table).
Grain proven rather than assumed: 65,752 distinct orders across 180,519 rows.

**Modelling**: role-playing date dimension (`USERELATIONSHIP` for order date vs shipping
date), `CALCULATE`-based conditional measures, iterators with scoped `VAR` blocks, what-if
parameters on a disconnected table.

**Security**: dynamic row-level security, one role plus a mapping table resolved with
`USERPRINCIPALNAME()`, tested with *View as* and *Test as role*.

**Reporting**: drillthrough with filter propagation, Q&A with a curated linguistic schema,
analytics reference lines, what-if interaction.

**Deployment**: published to a dedicated Power BI workspace, RLS members assigned,
semantic model endorsed as *Promoted*.

---

## Roadmap

* Bookmarks and button-driven navigation
* Key influencers and decomposition tree
* Date-continuous axis to unlock trend line and forecasting
* Migration of the Q&A page to Copilot (Power BI Q&A retires in December 2026)

---

*Dataset: [DataCo Smart Supply Chain](https://data.mendeley.com/datasets/8gx2fvg2k6/5), a public
research dataset. Built with Power BI Desktop and the Power BI Service.*
