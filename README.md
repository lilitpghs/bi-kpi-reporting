# Olist E-commerce: Sales & Delivery Performance Dashboard

A two-page Power BI dashboard analysing **99,000+ orders** from Olist, a Brazilian e-commerce marketplace. It answers two business questions:

1. **How is the business performing?** Revenue, orders, average order value and customer satisfaction over time, by state and by product category.
2. **How reliably do orders arrive?** On-time delivery, where delivery is slow, and which sellers are most often late.

![Overview page](figures/Overview.png)
![Delivery Performance page](figures/Delivery%20Performance.png)


---

## Key findings

| Metric | Value |
|---|---|
| Product revenue (excl. freight) | R$ 13.54M |
| Orders | 99,092 |
| Average order value | R$ 137.68 |
| Average review score | 4.09 / 5 |
| On-time delivery | 93.2% |
| Late orders | 6,532 |
| Average delivery time | 12.5 days |
| Average delay (late orders only) | 10.6 days |

**Sales**
- Revenue peaked in **November 2017** at R$ 1.01M, driven by Black Friday.
- **São Paulo (SP)** alone generates **38.3%** of revenue. The top 5 states (SP, RJ, MG, RS, PR) together account for about 74%.
- The top categories by revenue are **Health & Beauty, Watches & Gifts, Bed Bath & Table, Sports & Leisure and Computer Accessories**.
- In the latest month (Aug 2018), orders grew **+3.5%** while revenue fell **−4.6%**, so customers were buying more but cheaper orders.

**Delivery**
- On-time delivery is strong overall (93.2%) but fell **below 90% in three months**: Nov 2017, Feb 2018 and Mar 2018. The November drop shows the Black Friday peak strained logistics.
- Delivery time depends heavily on geography. Northern states such as **Roraima (29.9 days), Amapá (27.2) and Amazonas (26.4)** take more than twice the national average of 12.5 days.
- Among sellers with 50+ orders, the worst performers deliver only **70–79%** of orders on time. This makes them a clear, short list for seller-performance follow-up.

---

## Insights for decision-making

The dashboard is built for a marketplace manager or operations lead who needs to decide where to focus. Each insight below links what the data shows to a decision it supports.

1. Prepare logistics before peak season

What the data shows: Revenue peaked in November 2017 (Black Friday), and in the same month on-time delivery fell to 88%. It fell again to 86% and 81% in February–March 2018.
Decision it supports: Agree extra carrier capacity and warehouse staffing with logistics partners before November. Set more realistic estimated delivery dates during peak weeks so customers aren't promised dates that can't be met.

2. Set regional delivery promises and target the North

What the data shows: Northern states (RR, AP, AM, AL, PA) wait 24–30 days on average, about twice the national average of 12.5 days.
Decision it supports:

Show region-specific delivery estimates at checkout instead of one national promise.
Assess regional carriers or local fulfilment hubs for the North.
Treat these states as an expansion opportunity: faster delivery could unlock demand that slow shipping currently holds back.
3. Manage seller performance with clear thresholds

What the data shows: Most sellers deliver on time, but a small group with 50+ orders delivers only 70–79% on time, and some are 20–45 days late on average.
Decision it supports: Use the seller ranking as a monthly watch-list. Set a minimum on-time target (for example 90%), contact sellers below it, and apply consequences such as lower search ranking or a performance review if they don't improve. The list is short, so follow-up takes little effort.

4. Protect and grow the core markets

What the data shows: São Paulo produces 38% of revenue, and the top 5 states produce about 74%.
Decision it supports: Keep marketing spend and delivery quality highest in SP, RJ and MG, where most revenue is at stake. At the same time, the dependence on one state is a risk, which supports a deliberate plan to grow other regions.

5. Focus on the top categories

What the data shows: Health & Beauty, Watches & Gifts and Bed Bath & Table lead revenue.
Decision it supports: Prioritise these categories for seller recruitment, promotions and stock availability, especially ahead of peak season.

6. Watch order value, not only order volume

What the data shows: In August 2018 orders grew +3.5% but revenue fell −4.6%, so customers bought more but cheaper orders.
Decision it supports: Track average order value alongside order count. If the drop continues, test bundles, cross-selling or free-shipping thresholds to raise basket size.
---

How to use the dashboard
Monthly business review: the Overview page with its month-on-month indicators shows at a glance whether revenue and orders are moving in the right direction.
Operations meetings: the Delivery page shows which months, states and sellers need attention.
Drill-down: the Year and State slicers let a regional manager see the same KPIs for their own area.
---

## Dashboard pages

### 1. Overview
- KPI cards: revenue and orders, each with a month-on-month change (green/red), average order value, on-time delivery % and average review score
- Line chart of monthly product revenue, with the peak month highlighted
- Treemap of the top 5 states by revenue share
- Bar chart of the top 5 categories by revenue
- Slicers for year and customer state

### 2. Delivery Performance
- KPI cards: on-time delivery %, late orders, average delivery days and average days late
- Combo chart of on-time delivery % by purchase month, with an average line and months below 90% highlighted
- Bar chart of the slowest states, with a dynamic national-average reference line
- Table ranking sellers (50+ orders) by on-time delivery, with conditional formatting
- Page navigation buttons and synced slicers

---

## Dataset

[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (Kaggle).

Tables used: orders, order items, payments, customers, products, sellers and reviews. Product categories were translated from Portuguese to English.

---

## Data model

![Data model](figures/Model%20View.png)

- **`olist_orders_dataset`** is the central table, with one row per order.
- **Order items, payments and customers** link to orders on `order_id` / `customer_id`.
- **Products and sellers** link to order items.
- A dedicated **Date table** links to the order purchase date and is used for all time intelligence.
- Measures are organised in display folders: **Sales** and **Delivery**.

---

## Data preparation (Power Query)

- Fixed data types and date formats with the correct locale
- Merged review scores into the orders table
- Translated product category names to English and replaced missing categories with "Unknown"
- Created delivery columns: delivery days, delay days (actual vs estimated delivery date) and delivery status (On time / Late / Not delivered)
- Created shortened seller IDs for readable labels

---

## Selected DAX measures

**Average order value.** Only orders that contain items are counted, so cancelled or unavailable orders with no products don't pull the average down.
```dax
Orders With Items =
CALCULATE (
    COUNTROWS ( olist_orders_dataset ),
    NOT ISBLANK ( olist_orders_dataset[Items In Order] )
)

Average Order Value =
DIVIDE ( [Revenue], [Orders With Items] )
```

**On-time delivery %**
```dax
Delivered Orders =
CALCULATE ( COUNTROWS ( olist_orders_dataset ), olist_orders_dataset[Delivery Status] <> "Not delivered" )

Late Orders =
CALCULATE ( COUNTROWS ( olist_orders_dataset ), olist_orders_dataset[Delivery Status] = "Late" )

On-Time Delivery % =
DIVIDE ( [Delivered Orders] - [Late Orders], [Delivered Orders] )
```

**Month-on-month change for the latest month** (drives the green/red KPI subtitles)
```dax
Revenue MoM Latest % =
VAR LastDay   = MAX ( olist_orders_dataset[Purchase Date] )
VAR ThisMonth = FORMAT ( LastDay, "YYYY-MM" )
VAR PrevMonth = FORMAT ( EOMONTH ( LastDay, -1 ), "YYYY-MM" )
VAR Cur  = CALCULATE ( [Revenue], 'Date'[Year-Month] = ThisMonth )
VAR Prev = CALCULATE ( [Revenue], 'Date'[Year-Month] = PrevMonth )
RETURN DIVIDE ( Cur - Prev, Prev )
```

**Seller ranking** (only sellers with 50+ orders, so small sellers don't distort the ranking)
```dax
Seller Rank =
IF (
    [Orders Count] >= 50,
    RANKX (
        FILTER ( ALLSELECTED ( olist_sellers_dataset ), [Orders Count] >= 50 ),
        [On-Time Delivery %], , ASC
    )
)
```

**Dynamic chart title** (counts the months below target in the current selection)
```dax
On-Time Title =
VAR T    = ADDCOLUMNS ( ALLSELECTED ( 'Date'[Month Short] ), "Rate", [On-Time Delivery %] )
VAR Weak = COUNTROWS ( FILTER ( T, [Rate] < 0.9 ) )
RETURN
    IF ( Weak = 0,
        "On-time delivery stayed above 90% every month",
        "On-time delivery fell below 90% in " & Weak & IF ( Weak = 1, " month", " months" ) )
```

---

## Definitions & assumptions

- **Period:** orders purchased from **January 2017 to August 2018**. The first and last months of the raw data contain very few orders and would show a false collapse in revenue, so they are excluded.
- **Revenue** is the product price of order items. It **excludes freight** (another R$ 2.24M) and includes items from all order statuses.
- **On-time** means the order was delivered on or before the estimated delivery date. Undelivered orders are excluded from the on-time rate.
- **On-time delivery by month** groups orders by **purchase month**, not delivery month.
- **Average review score** includes only orders that received a review.
- Currency is **Brazilian Real (R$)**.

---

## Tools

- **Power BI Desktop:** Power Query, data modelling, DAX, report design
- **Power Query (M):** data cleaning and transformation
