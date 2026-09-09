# 📐 DAX Measures Reference

Full list of custom DAX measures built for this project, organized into display folders matching the Power BI model structure.

---

## 📁 Ungrouped / Helper Measures

```dax

  -- Avg Price
  (paste formula here)

```

---

## 📁 Customer KPIs

```dax
-- Customer Lifetime Value (CLV)
Customer Lifetime Value (CLV) = DIVIDE([Total Revenue], [Total Unique Customers])

-- New Customers
New Customers = 
CALCULATE(
    DISTINCTCOUNT(olist_customers_dataset[customer_unique_id]),
    FILTER(
        ALL(olist_orders_dataset),
        olist_orders_dataset[order_purchase_timestamp] = 
            CALCULATE(MIN(olist_orders_dataset[order_purchase_timestamp]), ALLEXCEPT(olist_customers_dataset, olist_customers_dataset[customer_unique_id]))
    )
)

-- Repeat Customers
Repeat Customers = 
COUNTROWS(
    FILTER(
        VALUES(olist_customers_dataset[customer_unique_id]),
        CALCULATE(DISTINCTCOUNT(olist_orders_dataset[order_id])) > 1
    )
)

-- Repeat Customer Rate
Repeat Customer Rate = DIVIDE([Repeat Customers], [Total Unique Customers])

-- Total Unique Customers
Total Unique Customers = DISTINCTCOUNT(olist_customers_dataset[customer_unique_id])
```

---

## 📁 Payment KPIs

```dax
-- Total Payment Value
Total Payment Value = SUM(olist_order_payments_dataset[payment_value])

-- Average Installments
Average Installments = AVERAGE(olist_order_payments_dataset[payment_installments])

-- % Credit Card Payments
% Credit Card Payments = DIVIDE(CALCULATE(COUNTROWS(olist_order_payments_dataset), olist_order_payments_dataset[payment_type]="credit_card"), COUNTROWS(olist_order_payments_dataset))
```

---

## 📁 Product Analytics

```dax
-- % Revenue by Category
% Revenue by Category = DIVIDE([Total Revenue], CALCULATE([Total Revenue], ALL(olist_products_dataset[product_category_name_english])))

-- Product Rank by Revenue
Product Rank by Revenue = RANKX(ALL(olist_products_dataset[product_id]), [Total Revenue], , DESC)

-- Avg Price
Avg Price = DIVIDE([Total Revenue], [Total Items Sold])
```

---

## 📁 Review & Logistics KPIs

```dax
-- Average Delivery Days
Average Delivery Days = 
AVERAGEX(
    FILTER(olist_orders_dataset, NOT(ISBLANK(olist_orders_dataset[order_delivered_customer_date]))),
    DATEDIFF(olist_orders_dataset[order_purchase_timestamp], olist_orders_dataset[order_delivered_customer_date], DAY)
)

-- Delivered Orders
Delivered Orders = CALCULATE([Total Orders], olist_orders_dataset[order_status] = "delivered")

-- On-Time Delivery %
On-Time Delivery % = 
VAR OnTime = CALCULATE(COUNTROWS(olist_orders_dataset), olist_orders_dataset[order_delivered_customer_date] <= olist_orders_dataset[order_estimated_delivery_date])
RETURN DIVIDE(OnTime, [Delivered Orders])

-- Average Review Score
Average Review Score = AVERAGE(olist_order_reviews_dataset[review_score])

-- % 5-Star Reviews
% 5-Star Reviews = DIVIDE(CALCULATE(COUNTROWS(olist_order_reviews_dataset), olist_order_reviews_dataset[review_score]=5), COUNTROWS(olist_order_reviews_dataset))

-- Avg Review Score - Late Delivery
Avg Review Score - Late Delivery = 
CALCULATE([Average Review Score], olist_orders_dataset[order_delivered_customer_date] > olist_orders_dataset[order_estimated_delivery_date])

-- Avg Review Score - On Time
Avg Review Score - On Time = 
CALCULATE([Average Review Score], olist_orders_dataset[order_delivered_customer_date] <= olist_orders_dataset[order_estimated_delivery_date])

-- % Late Delivery
% Late Delivery = 1 - [On-Time Delivery %]

-- Avg Review Score by Category
Avg Review Score by Category = 
CALCULATE(
    [Average Review Score],
    CROSSFILTER(olist_order_items_dataset[order_id], olist_orders_dataset[order_id], Both)
)
```

---

## 📁 Sales KPIs

```dax
-- Total Revenue
Total Revenue = SUMX(olist_order_items_dataset, olist_order_items_dataset[price] + olist_order_items_dataset[freight_value])

-- Total Orders
Total Orders = DISTINCTCOUNT(olist_orders_dataset[order_id])

-- Total Items Sold
Total Items Sold = COUNTROWS(olist_order_items_dataset)

-- Average Order Value
Average Order Value = DIVIDE([Total Revenue], [Total Orders])

-- Total Freight
Total Freight = SUM(olist_order_items_dataset[freight_value])

-- Freight % of Revenue
Freight % of Revenue = DIVIDE([Total Freight], [Total Revenue])
```

---

## 📁 Time Intelligence

```dax
-- Revenue MoM Growth
Revenue MoM Growth = 
    VAR CurrentRevenue = [Total Revenue]
    VAR PreviousRevenue = CALCULATE([Total Revenue], DATEADD(DateTable[Date], -1, MONTH))
        RETURN DIVIDE(CurrentRevenue - PreviousRevenue, PreviousRevenue)

-- Revenue MTD
Revenue MTD = TOTALMTD([Total Revenue], DateTable[Date])

-- Revenue QTD
Revenue QTD = TOTALQTD([Total Revenue], DateTable[Date])

-- Revenue YTD
Revenue YTD = TOTALYTD([Total Revenue], DateTable[Date])

-- Revenue YoY Growth
Revenue YoY Growth = 
VAR CurrentRevenue = [Total Revenue]
VAR PreviousRevenue = CALCULATE([Total Revenue], SAMEPERIODLASTYEAR(DateTable[Date]))
RETURN DIVIDE(CurrentRevenue - PreviousRevenue, PreviousRevenue)
```

---

⬅️ [Back to main README](../README.md)
