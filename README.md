# Northwind Traders: Executive KPI Dashboard

A two-page Power BI dashboard that shows the Northwind executive team how the business is performing this year, and what to do about it.

![Overview page](images/01-overview.png)

**Files:** [Power BI file](Northwind_KPI_Dashboard.pbix) | [PDF export](Northwind_KPI_Dashboard.pdf)

---

## The brief

Northwind Traders is a fictional company that sells gourmet food to restaurants and shops around the world. The task was to build a top-level KPI dashboard for the executive team covering four areas: sales trends, product performance, key customers and shipping costs. The audience is senior management, so I kept it to two pages. Page 1 answers "how are we doing this year?" in one glance. Page 2 shows which products, customers and carriers drive the numbers, with my insights and recommendations.

---

## Key insights

|     | Insight                                    | Evidence                                                                                                                                         |
| --- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1   | **Growth is volume-led**                   | Revenue is up 119.4% year to date. Orders are up 109.3%, but average order value only rose 4.8%. Only 1 of 81 active customers this year is new. |
| 2   | **Revenue is concentrated**                | The top 3 customers bring in 25.2% of all revenue. Losing any one of them would cut about 8% of revenue.                                         |
| 3   | **One product carries the range**          | Côte de Blaye is 15.3% of 2015 revenue on its own. Discontinued products still earn 14.1%.                                                       |
| 4   | **Freight is growing faster than revenue** | Freight rose from 4.5% to 5.0% of revenue. United Package handles 43% of orders but 55% of freight spend.                                        |
|     |                                            |                                                                                                                                                  |

![Details page](images/02-details.png)

### Recommendations

1. **Renegotiate United Package rates or move volume to Speedy Express.** Speedy Express costs $65.00 per order against $86.64, and it ships just as fast.
2. **Assign account managers to the top 10 customers** and set a target for winning new ones, so growth does not depend on a few existing buyers.
3. **Plan replacements for discontinued products that still sell,** and secure the supply of Côte de Blaye.

---

## Data and model

The dataset has 7 CSV tables with 2,985 records: orders, order details, products, categories, customers, employees and shippers. The orders run from 4 July 2013 to 6 May 2015.

I built a star schema with `orders` and `order_details` as the fact tables and the other tables as dimensions. All relationships are one-to-many and single direction. I also added a date table that covers full calendar years (2013 to 2015), marked it as a date table, and turned off Power BI's automatic date tables.

![Data model](images/03-data-model.png)

---

## Key decisions

**Comparing year to date, not full years.** The data starts in July 2013 and ends in May 2015, so both years are incomplete. A full-year comparison would show 2015 down 29%, when the business actually grew 119%. Every KPI compares 1 Jan to 6 May 2015 with the same window in 2014.

**Capping last year at 6 May.** My date table runs to 31 December 2015, so a standard "same period last year" measure would compare four months of 2015 against all of 2014. I capped the prior-year window at the last order date minus one year:

```dax
Revenue PY YTD =
VAR CutoffPY = EDATE ( [Last Order Date], -12 )
RETURN
    CALCULATE (
        [Revenue],
        SAMEPERIODLASTYEAR ( DATESYTD ( 'Date'[Date] ) ),
        'Date'[Date] <= CutoffPY
    )
```

The limitation: this cap is tied to the latest date in the data, so it is only correct when looking at the current year. That fits an executive "this year" view, but it would need changing for a year slicer.

**Net revenue from the price actually charged.** 658 order lines were sold at a different price from the product's current list price. Revenue uses the price on the order line, after discount, and is calculated row by row:

```dax
Revenue =
SUMX (
    order_details,
    order_details[quantity] * order_details[unitPrice] * ( 1 - order_details[discount] )
)
```

**Freight counted per order.** Freight is stored once per order. Summing it through the order lines would count it several times, so all freight measures sum the `orders` table directly.

---

## Problems I solved

**Broken product and customer names.** Names like "Côte de Blaye" and "Königlich Essen" showed as "C�te de Blaye". Two of the CSV files were saved in Windows-1252 encoding, but Power BI read them as UTF-8. I changed the encoding in the Power Query source step (`Encoding=1252`), which fixed every accented name.

---

## Tools

- **Power BI Desktop:** data model and report
- **DAX:** over 20 measures, including year-to-date comparisons and conditional formatting
- **Power Query:** data import and encoding fix

---

## What I would add next

- **On-time delivery metrics** using shipped and required dates, to compare carriers on service as well as cost
- **An employee performance page** using the sales rep data
- **A year slicer,** which would need the prior-year cap reworked

---

*Built by Alex Antony. Feedback welcome.*
