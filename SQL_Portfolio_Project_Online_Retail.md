# Online Retail Revenue and Customer Analysis
**SQL Portfolio Project — Zohaib Muallim**

## Overview

I analysed a UK based online retail transaction dataset covering December 2010 to late 2011 using SQL, to understand revenue distribution, product performance, seasonal trends and customer churn risk.

Tool used: SQLite (browser based SQL environment)
Dataset: Online Retail transaction data (invoices, products, quantities, prices, customers, countries)

---

## 1. Total Revenue by Country

```sql
SELECT
    c8 AS Country,
    SUM(c4 * c6) AS TotalRevenue
FROM data
WHERE c4 > 0
GROUP BY c8
ORDER BY TotalRevenue DESC;
```

**Finding:** Revenue is heavily concentrated in the UK market, which generated 8.2 million pounds, more than 28 times the next largest market, the Netherlands at 285 thousand pounds. This shows the business is almost entirely domestic despite having customers across multiple countries.

---

## 2. Top 10 Products by Revenue

```sql
SELECT
    c3 AS Product,
    SUM(c4 * c6) AS ProductRevenue
FROM data
WHERE c4 > 0
GROUP BY c3
ORDER BY ProductRevenue DESC
LIMIT 10;
```

**Finding:** Postage and shipping charges were being counted as a top revenue line, which would have distorted a straightforward top products report if not caught and flagged separately. Once identified, the actual top selling product was the Regency Cakestand 3 Tier at 165 thousand pounds, followed by seasonal and gift items such as heart shaped tea light holders and party bunting.

---

## 3. Monthly Revenue Trend

```sql
SELECT
    CAST(substr(substr(c5, instr(c5,'/')+1), instr(substr(c5, instr(c5,'/')+1), '/')+1, 4) AS INTEGER) AS Year,
    CAST(substr(c5, 1, instr(c5, '/') - 1) AS INTEGER) AS Month,
    SUM(c4 * c6) AS MonthlyRevenue
FROM data
WHERE c4 > 0
GROUP BY Year, Month
ORDER BY Year, Month;
```

**Finding:** The monthly revenue trend showed a dip in the new year following the December peak, before recovering and climbing through spring into summer, suggesting a seasonal pattern worth planning stock and marketing around.

---

## 4. Customer Churn Risk

```sql
SELECT
    c7 AS CustomerID,
    MAX(c5) AS LastPurchaseRaw,
    (
        CAST(substr(substr(c5, instr(c5,'/')+1), instr(substr(c5, instr(c5,'/')+1), '/')+1, 4) AS INTEGER) * 10000
        + CAST(substr(c5, 1, instr(c5, '/') - 1) AS INTEGER) * 100
        + CAST(substr(substr(c5, instr(c5,'/')+1), 1, instr(substr(c5, instr(c5,'/')+1), '/')-1) AS INTEGER)
    ) AS SortableDate,
    CASE
        WHEN (
            CAST(substr(substr(c5, instr(c5,'/')+1), instr(substr(c5, instr(c5,'/')+1), '/')+1, 4) AS INTEGER) * 10000
            + CAST(substr(c5, 1, instr(c5, '/') - 1) AS INTEGER) * 100
            + CAST(substr(substr(c5, instr(c5,'/')+1), 1, instr(substr(c5, instr(c5,'/')+1), '/')-1) AS INTEGER)
        ) < 20110601
        THEN 'At Risk'
        ELSE 'Active'
    END AS Status
FROM data
WHERE c4 > 0 AND c7 IS NOT NULL AND c7 != 'CustomerID'
GROUP BY c7
ORDER BY SortableDate ASC
LIMIT 20;
```

**Finding:** Flagging customers whose most recent purchase fell before June 2011 as at risk identified a clear group of early customers, largely from December 2010, who had not returned, representing a concrete group the business could target with a win back campaign.

---

## Summary

This analysis combined filtering, aggregation, grouping, sorting, text based date parsing and conditional logic (CASE statements) to move from raw transaction data to four practical business findings: where revenue comes from, what actually sells best, how demand shifts across the year, and which customers are at risk of being lost. Each finding points to a concrete next step a business could act on, from prioritising the UK market to planning a customer win back campaign.
