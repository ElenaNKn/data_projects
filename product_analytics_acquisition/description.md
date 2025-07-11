# Acquisition Metrics Dashboard

This project uses data from the ["SQL simulator"](https://karpov.courses/simulator-sql).
The goal is to build a dashboard to track acquisition metrics of a delivery service.
The database schema and tables' structure are described [here](database_structure.md)

## Tracked Metrics

The dashboard includes the following daily metrics:
1. number of new users and new couriers added that day;
2. current (total) number of users and couriers;
3. change in new users and new couriers (compared to the previous day);
4. change in total users and total couriers (compared to the previous day);
5. number of paying users and active couriers;
6. paying users share and active couriers share within current users and current couriers counts accordingly;
7. share of users who created only one and more one order within the day
8. number of orders, first orders and orders of new users
9. shares of first-time orders and new-user orders in daily order volume
10. number of paying users and orders per active couriers
11. average time to deliver order
12. success orders, canceled orders and orders cancel rate by hours

## SQL Queries for Metric Calculations in PostgreSQL

<b>Metrics 1-7</b>

```sql
WITH users_count 
AS 
(
    SELECT start_date AS date
        ,COUNT(*)::INT AS new_users
        ,SUM(COUNT(*)) OVER (ORDER BY start_date)::INT AS total_users
    FROM   
    (
        SELECT user_id
            ,DATE(MIN(time)) as start_date    -- date of the first user action
        FROM   user_actions
        GROUP BY user_id
    ) AS new_users
    GROUP BY date
), 
couriers_count 
AS 
(
    SELECT start_date AS date
        ,COUNT(*)::int AS new_couriers
        ,SUM(COUNT(*)) OVER (ORDER BY start_date)::INT AS total_couriers
    FROM   
    (
        SELECT courier_id
            ,DATE(MIN(time)) AS start_date    -- date of the first user action
        FROM courier_actions
        GROUP BY courier_id
    ) AS new_couriers
GROUP BY date
)
SELECT 
    date
    ,new_users
    ,new_couriers
    ,total_users
    ,paying_users
    ,total_couriers
    ,active_couriers
    ,ROUND(100 * (new_users - prev_new_users) / (prev_new_users::decimal), 2) AS new_users_change
    ,ROUND(100 * (total_users - prev_users) / prev_users::decimal, 2) AS total_users_growth
    ,ROUND(100 * (new_couriers - prev_new_couriers) / prev_new_couriers::DECIMAL, 2) AS new_couriers_change
    ,ROUND(100 * (total_couriers - prev_couriers) / prev_couriers::DECIMAL, 2) AS total_couriers_growth
    ,ROUND(100 * paying_users / total_users::DECIMAL, 2) AS paying_users_share
    ,ROUND(100 * active_couriers / total_couriers::DECIMAL, 2) AS active_couriers_share
    ,ROUND(100 * single_order_users / paying_users::DECIMAL, 2) AS single_order_users_share
    ,ROUND(100 * several_orders_users / paying_users::DECIMAL, 2) AS several_orders_users_share
FROM   
(
    SELECT users_count.date AS date
        ,new_users
        ,new_couriers
        ,total_users
        ,paying_users
        ,single_order_users
        ,several_orders_users
        ,total_couriers
        ,active_couriers
        ,lag(new_users) OVER (ORDER BY users_count.date) AS prev_new_users
        ,lag(new_couriers) OVER (ORDER BY users_count.date) AS prev_new_couriers
        ,lag(total_users) OVER (ORDER BY users_count.date) AS prev_users
        ,lag(total_couriers) OVER (ORDER BY users_count.date) AS prev_couriers
    FROM users_count
    LEFT JOIN (
                SELECT date
                    ,COUNT(user_id) AS paying_users
                    ,SUM(CASE WHEN orders_count = 1 THEN 1 ELSE 0 END) AS single_order_users
                    ,SUM(CASE WHEN orders_count > 1 THEN 1 ELSE 0 END) AS several_orders_users
                FROM
                (
                    SELECT DATE(time) AS date
                        ,user_id
                        ,COUNT(DISTINCT order_id) AS orders_count
                    FROM user_actions 
                    -- exclude canceled orders
                    WHERE order_id NOT IN (SELECT order_id FROM user_actions WHERE action = 'cancel_order')
                    GROUP BY date, user_id
                ) AS a
                GROUP BY date
            ) AS paying
    ON users_count.date = paying.date
    LEFT JOIN couriers_count 
    ON users_count.date = couriers_count.date
    LEFT JOIN (
                SELECT DATE(time) AS date
                    ,COUNT(DISTINCT courier_id) AS active_couriers
                FROM courier_actions 
                -- exclude canceled orders
                WHERE order_id NOT IN (SELECT order_id FROM user_actions WHERE action = 'cancel_order')
                GROUP BY date
            ) AS act_couriers
    ON users_count.date = act_couriers.date
) AS agg
ORDER BY date
```

<b>Metrics 8-9</b>

```sql
SELECT *
    -- share of the first orders in total orders
    ,ROUND(100 * first_orders::DECIMAL / orders, 2) AS first_orders_share
    -- share of new users' orders in total orders
    ,ROUND(100 * new_users_orders::DECIMAL / orders, 2) AS new_users_orders_share
FROM
(
    SELECT date
        ,SUM(orders_count)::INT AS orders
        --count of first orders is equal to count of new users
        ,COUNT(*) FILTER (WHERE date = first_order_user_date) AS first_orders
        ,SUM(orders_count) FILTER (WHERE date = start_user_date)::INT AS new_users_orders
    FROM
    (
        SELECT DISTINCT DATE(time) AS date
            ,user_id
            ,COUNT(order_id) 
                FILTER (WHERE order_id NOT IN (SELECT order_id FROM user_actions WHERE action = 'cancel_order'))
                OVER (PARTITION BY user_id, DATE(time)) AS orders_count
            -- start date of users life
            ,MIN(DATE(time)) OVER (PARTITION BY user_id) AS start_user_date
            -- date of the first order
            ,MIN(DATE(time)) 
                FILTER (WHERE order_id NOT IN (SELECT order_id FROM user_actions WHERE action = 'cancel_order')) 
                OVER (PARTITION BY user_id) AS first_order_user_date
        FROM user_actions 
    ) AS a
    GROUP BY date
) AS b
ORDER BY date
```

<b>Metric 10</b>

```sql
SELECT paying.date AS date
    ,ROUND(paying_users::DECIMAL / active_couriers, 2) AS users_per_courier
    ,ROUND(day_orders::DECIMAL / active_couriers, 2) AS orders_per_courier
FROM
(
    SELECT date
        ,COUNT(user_id) AS paying_users
        ,SUM(orders_count) AS day_orders
    FROM
    (
        SELECT DATE(time) AS date
            ,user_id
            ,COUNT(DISTINCT order_id) AS orders_count
        FROM user_actions 
        WHERE order_id NOT IN (SELECT order_id FROM user_actions WHERE action = 'cancel_order')
        GROUP BY date, user_id
    ) AS a
    GROUP BY date
) AS paying
LEFT JOIN 
(
    SELECT DATE(time) AS date
        ,COUNT(DISTINCT courier_id) AS active_couriers
    FROM courier_actions 
    WHERE order_id NOT IN (SELECT order_id FROM user_actions WHERE action = 'cancel_order')
    GROUP BY date
) AS act_couriers
ON paying.date = act_couriers.date
ORDER BY date
```

<b>Metric 11</b>

```sql
SELECT date
    ,AVG(order_minutes) FILTER (WHERE order_minutes > 0)::INT AS minutes_to_deliver
FROM
(
    SELECT courier_id
        ,order_id
        ,DATE(MAX(time)) AS date
        ,ROUND(EXTRACT(EPOCH FROM (MAX(time) - MIN(time))) / 60) AS order_minutes
    FROM courier_actions 
    WHERE order_id NOT IN (SELECT order_id FROM user_actions WHERE action = 'cancel_order')
    GROUP BY courier_id, order_id
) AS a
GROUP BY date
ORDER BY date
```

<b>Metric 12</b>

```sql
SELECT hour::INTEGER
    ,COUNT(*) - SUM(is_canceled) AS successful_orders
    ,SUM(is_canceled) AS canceled_orders
    ,ROUND(SUM(is_canceled)::DECIMAL / COUNT(*), 3) AS cancel_rate
FROM
(
    SELECT EXTRACT (HOUR FROM creation_time) AS hour
        ,CASE WHEN order_id IN (SELECT DISTINCT order_id 
                                FROM user_actions 
                                WHERE action='cancel_order') 
                        THEN 1 ELSE 0 END AS is_canceled
    FROM orders
) AS a
GROUP BY hour
ORDER BY hour
```

## Tableau Dashboard
The dashboard is uploaded at Tableau Public: https://public.tableau.com/views/Acquisitionmetricsofdeliveryservice/Dashboard