# Product Metrics Dashboard

This project uses data from the ["SQL simulator"](https://karpov.courses/simulator-sql).
The goal is to build a dashboard to track product metrics of a delivery service.
The database schema and tables' structure are described [here](database_structure.md)

## Tracked Metrics

The dashboard includes the following metrics:
1. Daily Revenue and Change of Daily Revenue by days; 
2. Total Revenue and Total Costs (cumulative values by days), calculated with different tax coefficients for multiple products;
3. Daily Gross Profit and Gross Profit Ratio:
4. Total Gross Profit and Total Gross Profit Ratio (cumulative values of daily metrics);
5. ARPPPU (Average Revenue per Paying User), ARPU (Average Revenue per User) and AOV (Average Order value) dynamics by Days;
6. ARPPPU (Average Revenue per Paying User), ARPU (Average Revenue per User) and AOV (Average Order value) distribution by Weekdays;
7. Running ARPPU, ARPU and AOV by Day (cumulative values by days);
8. Revenue from New Users (showing the share of revenue from new and existing users);
9. Revenue by Products.

## SQL Queries for Metric Calculations in PostgreSQL

<b>Metric 1</b>

```sql
SELECT *
    ,ROUND(100 * (revenue - lag(revenue) OVER (ORDER BY date)) / lag(revenue) OVER (ORDER BY date), 2) AS revenue_change
FROM
(
    SELECT date
        ,revenue
        ,SUM(revenue) OVER (ORDER BY date) AS total_revenue
    FROM
    (
        SELECT date
            ,SUM(price) AS revenue
        FROM
        (
            SELECT DATE(creation_time) AS date
                ,unnest(product_ids) AS product
            FROM orders
            WHERE order_id NOT IN (SELECT DISTINCT order_id FROM user_actions WHERE action = 'cancel_order')
        ) AS a
        JOIN products 
        ON a.product = products.product_id
        GROUP BY date
    ) AS a
) AS a
ORDER BY date
```

<b>Metrics 2 - 4</b>

```sql
WITH cost1 AS 
(
    SELECT date
        ,CASE WHEN DATE_TRUNC('month', date) = '2022-08-01' THEN 
            120000 + 140 * order_gether
            ELSE 150000 + 115 * order_gether
            END AS cost1
    FROM
    (
        SELECT DATE(creation_time) AS date
            ,COUNT(DISTINCT order_id) AS order_gether
        FROM orders 
        WHERE order_id NOT IN (SELECT DISTINCT order_id FROM user_actions WHERE action = 'cancel_order')
        GROUP BY date
    ) AS a
),

cost2 AS 
(
    SELECT date
        ,CASE WHEN DATE_TRUNC('month', date) = '2022-08-01' THEN 
                150 * SUM(orders_count) + 400 * COUNT(courier_id) FILTER (WHERE orders_count >= 5)
                ELSE 150 * SUM(orders_count) + 500 * COUNT(courier_id) FILTER (WHERE orders_count >= 5)
                END AS cost2
    FROM
    (
        SELECT DATE(time) AS date
            ,courier_id
            ,COUNT(DISTINCT order_id) AS orders_count
        FROM courier_actions
        WHERE action = 'deliver_order'
        GROUP BY date, courier_id
    ) AS a
    GROUP BY date
),

revenue AS
(
    SELECT date
        ,SUM(price) AS revenue
        ,SUM(ROUND(CASE WHEN 
            name IN ('сахар', 'сухарики', 'сушки', 'семечки', 
                    'масло льняное', 'виноград', 'масло оливковое', 
                    'арбуз', 'батон', 'йогурт', 'сливки', 'гречка', 
                    'овсянка', 'макароны', 'баранина', 'апельсины', 
                    'бублики', 'хлеб', 'горох', 'сметана', 'рыба копченая', 
                    'мука', 'шпроты', 'сосиски', 'свинина', 'рис', 
                    'масло кунжутное', 'сгущенка', 'ананас', 'говядина', 
                    'соль', 'рыба вяленая', 'масло подсолнечное', 'яблоки', 
                    'груши', 'лепешка', 'молоко', 'курица', 'лаваш', 'вафли', 'мандарины')
            THEN price * 0.1/(1 + 0.1) ELSE price * 0.2/(1 + 0.2) END, 2)) AS tax
    FROM
    (
        SELECT DATE(creation_time) AS date
            ,unnest(product_ids) AS product
        FROM orders
        WHERE order_id NOT IN (SELECT DISTINCT order_id FROM user_actions WHERE action = 'cancel_order')
    ) AS a
    JOIN products 
    ON a.product = products.product_id
    GROUP BY date
)

SELECT *
    ,SUM(revenue) OVER(ORDER BY date) AS total_revenue
    ,SUM(costs) OVER(ORDER BY date) AS total_costs
    ,SUM(tax) OVER(ORDER BY date) AS total_tax
    ,SUM(gross_profit) OVER(ORDER BY date)  AS total_gross_profit
    ,ROUND(100 * gross_profit / revenue::DECIMAL, 2) AS gross_profit_ratio
    ,ROUND(100 * (SUM(gross_profit) OVER(ORDER BY date)) / (SUM(revenue) OVER(ORDER BY date))::DECIMAL, 2) AS total_gross_profit_ratio
FROM
(
    SELECT revenue.date AS date
        ,revenue
        ,cost1 + cost2 AS costs
        ,tax
        ,revenue - (cost1 + cost2) - tax AS gross_profit
    FROM revenue
    JOIN cost1
    USING(date)
    JOIN cost2
    USING(date)
) AS a
ORDER BY date
```