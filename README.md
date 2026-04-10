-- ============================================================
--  E-Commerce Sales Analysis — SQL Queries
--  Author : Diveshika Yadav
--  GitHub : github.com/diveshikayadav12
--  Email  : diveshika.yadav.cs28@iilm.edu
-- ============================================================

CREATE TABLE IF NOT EXISTS ecommerce_sales (
    order_id      INTEGER PRIMARY KEY,
    month         TEXT,
    city          TEXT,
    category      TEXT,
    product       TEXT,
    quantity      INTEGER,
    unit_price    REAL,
    discount      REAL,
    final_revenue REAL,
    payment_mode  TEXT,
    order_status  TEXT,
    rating        REAL
);

-- Q1: Overall Business KPIs
SELECT
    COUNT(*)                              AS total_orders,
    ROUND(SUM(final_revenue), 0)          AS total_revenue,
    ROUND(AVG(final_revenue), 0)          AS avg_order_value,
    SUM(quantity)                         AS total_units,
    ROUND(AVG(rating), 2)                 AS avg_rating
FROM ecommerce_sales
WHERE order_status = 'Delivered';

-- Q2: Revenue by City (Top Cities)
SELECT
    city,
    ROUND(SUM(final_revenue), 0)                                   AS revenue,
    COUNT(*)                                                        AS orders,
    ROUND(100.0 * SUM(final_revenue) /
          (SELECT SUM(final_revenue) FROM ecommerce_sales
           WHERE order_status='Delivered'), 1)                      AS revenue_pct
FROM ecommerce_sales
WHERE order_status = 'Delivered'
GROUP BY city
ORDER BY revenue DESC;

-- Q3: Category Performance
SELECT
    category,
    ROUND(SUM(final_revenue), 0)    AS revenue,
    SUM(quantity)                   AS units_sold,
    ROUND(AVG(discount)*100, 1)     AS avg_discount_pct,
    ROUND(AVG(rating), 2)           AS avg_rating
FROM ecommerce_sales
WHERE order_status = 'Delivered'
GROUP BY category
ORDER BY revenue DESC;

-- Q4: Top 10 Products
SELECT
    product,
    category,
    ROUND(SUM(final_revenue), 0)    AS revenue,
    SUM(quantity)                   AS units_sold
FROM ecommerce_sales
WHERE order_status = 'Delivered'
GROUP BY product, category
ORDER BY revenue DESC
LIMIT 10;

-- Q5: Monthly Revenue Trend
SELECT
    month,
    ROUND(SUM(final_revenue), 0)    AS monthly_revenue,
    COUNT(*)                        AS orders
FROM ecommerce_sales
WHERE order_status = 'Delivered'
GROUP BY month
ORDER BY CASE month
    WHEN 'Jan' THEN 1  WHEN 'Feb' THEN 2  WHEN 'Mar' THEN 3
    WHEN 'Apr' THEN 4  WHEN 'May' THEN 5  WHEN 'Jun' THEN 6
    WHEN 'Jul' THEN 7  WHEN 'Aug' THEN 8  WHEN 'Sep' THEN 9
    WHEN 'Oct' THEN 10 WHEN 'Nov' THEN 11 WHEN 'Dec' THEN 12
END;

-- Q6: Order Status Distribution
SELECT
    order_status,
    COUNT(*)                                              AS count,
    ROUND(100.0 * COUNT(*) / (SELECT COUNT(*) FROM ecommerce_sales), 1) AS percentage
FROM ecommerce_sales
GROUP BY order_status
ORDER BY count DESC;

-- Q7: Payment Mode Analysis
SELECT
    payment_mode,
    COUNT(*)                         AS transactions,
    ROUND(SUM(final_revenue), 0)     AS revenue
FROM ecommerce_sales
WHERE order_status = 'Delivered'
GROUP BY payment_mode
ORDER BY revenue DESC;

-- Q8: City x Category Revenue Matrix (Pivot)
SELECT
    city,
    ROUND(SUM(CASE WHEN category='Electronics'      THEN final_revenue ELSE 0 END), 0) AS Electronics,
    ROUND(SUM(CASE WHEN category='Clothing'         THEN final_revenue ELSE 0 END), 0) AS Clothing,
    ROUND(SUM(CASE WHEN category='Food & Grocery'   THEN final_revenue ELSE 0 END), 0) AS Food_Grocery,
    ROUND(SUM(CASE WHEN category='Furniture'        THEN final_revenue ELSE 0 END), 0) AS Furniture,
    ROUND(SUM(CASE WHEN category='Sports & Fitness' THEN final_revenue ELSE 0 END), 0) AS Sports_Fitness
FROM ecommerce_sales
WHERE order_status = 'Delivered'
GROUP BY city
ORDER BY city;

-- Q9: High Value Orders (above average)
SELECT order_id, product, city, quantity, ROUND(final_revenue,0) AS revenue
FROM ecommerce_sales
WHERE final_revenue > (SELECT AVG(final_revenue) FROM ecommerce_sales)
  AND order_status = 'Delivered'
ORDER BY revenue DESC
LIMIT 15;

-- Q10: Low Rating Products (needs improvement)
SELECT
    product,
    category,
    ROUND(AVG(rating), 2) AS avg_rating,
    COUNT(*)              AS reviews
FROM ecommerce_sales
GROUP BY product, category
HAVING avg_rating < 3.5
ORDER BY avg_rating ASC;
