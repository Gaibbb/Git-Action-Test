---
title: SQL 聚合
tags: []
id: '18'
categories:
  - - 'Linux'
date: 2023-08-26 12:56:00
---

# 聚合函数
## 表
```sql
-- 表invoices
+------------+-------------+-----------+---------------+---------------+--------------+------------+--------------+
| invoice_id | number      | client_id | invoice_total | payment_total | invoice_date | due_date   | payment_date |
+------------+-------------+-----------+---------------+---------------+--------------+------------+--------------+
|          1 | 91-953-3396 |         2 |        101.79 |          0.00 | 2019-03-09   | 2019-03-29 | NULL         |
|          2 | 03-898-6735 |         5 |        175.32 |          8.18 | 2019-06-11   | 2019-07-01 | 2019-02-12   |
|          3 | 20-228-0335 |         5 |        147.99 |          0.00 | 2019-07-31   | 2019-08-20 | NULL         |
|          4 | 56-934-0748 |         3 |        152.21 |          0.00 | 2019-03-08   | 2019-03-28 | NULL         |
|          5 | 87-052-3121 |         5 |        169.36 |          0.00 | 2019-07-18   | 2019-08-07 | NULL         |
|          6 | 75-587-6626 |         1 |        157.78 |         74.55 | 2019-01-29   | 2019-02-18 | 2019-01-03   |
|          7 | 68-093-9863 |         3 |        133.87 |          0.00 | 2019-09-04   | 2019-09-24 | NULL         |
|          8 | 78-145-1093 |         1 |        189.12 |          0.00 | 2019-05-20   | 2019-06-09 | NULL         |
|          9 | 77-593-0081 |         5 |        172.17 |          0.00 | 2019-07-09   | 2019-07-29 | NULL         |
|         10 | 48-266-1517 |         1 |        159.50 |          0.00 | 2019-06-30   | 2019-07-20 | NULL         |
|         11 | 20-848-0181 |         3 |        126.15 |          0.03 | 2019-01-07   | 2019-01-27 | 2019-01-11   |
|         13 | 41-666-1035 |         5 |        135.01 |         87.44 | 2019-06-25   | 2019-07-15 | 2019-01-26   |
|         15 | 55-105-9605 |         3 |        167.29 |         80.31 | 2019-11-25   | 2019-12-15 | 2019-01-15   |
|         16 | 10-451-8824 |         1 |        162.02 |          0.00 | 2019-03-30   | 2019-04-19 | NULL         |
|         17 | 33-615-4694 |         3 |        126.38 |         68.10 | 2019-07-30   | 2019-08-19 | 2019-01-15   |
|         18 | 52-269-9803 |         5 |        180.17 |         42.77 | 2019-05-23   | 2019-06-12 | 2019-01-08   |
|         19 | 83-559-4105 |         1 |        134.47 |          0.00 | 2019-11-23   | 2019-12-13 | NULL         |
+------------+-------------+-----------+---------------+---------------+--------------+------------+--------------+

-- 表payments
+------------+-----------+------------+------------+--------+----------------+
| payment_id | client_id | invoice_id | date       | amount | payment_method |
+------------+-----------+------------+------------+--------+----------------+
|          1 |         5 |          2 | 2019-02-12 |   8.18 |              1 |
|          2 |         1 |          6 | 2019-01-03 |  74.55 |              1 |
|          3 |         3 |         11 | 2019-01-11 |   0.03 |              1 |
|          4 |         5 |         13 | 2019-01-26 |  87.44 |              1 |
|          5 |         3 |         15 | 2019-01-15 |  80.31 |              1 |
|          6 |         3 |         17 | 2019-01-15 |  68.10 |              1 |
|          7 |         5 |         18 | 2019-01-08 |  32.77 |              1 |
|          8 |         5 |         18 | 2019-01-08 |  10.00 |              2 |
+------------+-----------+------------+------------+--------+----------------+

-- 表payment_methods
+-------------------+---------------+
| payment_method_id | name          |
+-------------------+---------------+
|                 1 | Credit Card   |
|                 2 | Cash          |
|                 3 | PayPal        |
|                 4 | Wire Transfer |
+-------------------+---------------+

-- 表orders
+----------+-------------+------------+--------+------------+
| order_id | customer_id | order_date | status | shipper_id |
+----------+-------------+------------+--------+------------+
|        1 |           6 | 2019-01-30 |      1 |       NULL |
|        2 |           7 | 2018-08-02 |      2 |          4 |
|        3 |           8 | 2017-12-01 |      1 |       NULL |
|        4 |           2 | 2017-01-22 |      1 |       NULL |
|        5 |           5 | 2017-08-25 |      2 |          3 |
|        6 |          10 | 2018-11-18 |      1 |       NULL |
|        7 |           2 | 2018-09-22 |      2 |          4 |
|        8 |           5 | 2018-06-08 |      1 |       NULL |
|        9 |          10 | 2017-07-05 |      2 |          1 |
|       10 |           6 | 2018-04-22 |      2 |          2 |
+----------+-------------+------------+--------+------------+

-- 表customers
+-------------+------------+------------+------------+--------------+------------------------+------------------+-------+--------+
| customer_id | first_name | last_name  | birth_date | phone        | address                | city             | state | points |
+-------------+------------+------------+------------+--------------+------------------------+------------------+-------+--------+
|           1 | Babara     | MacCaffrey | 1986-03-28 | 781-932-9754 | 0 Sage Terrace         | Waltham          | MA    |   2273 |
|           2 | Ines       | Brushfield | 1986-04-13 | 804-427-9456 | 14187 Commercial Trail | Hampton          | VA    |    947 |
|           3 | Freddi     | Boagey     | 1985-02-07 | 719-724-7869 | 251 Springs Junction   | Colorado Springs | CO    |   2967 |
|           4 | Ambur      | Roseburgh  | 1974-04-14 | 407-231-8017 | 30 Arapahoe Terrace    | Orlando          | FL    |    457 |
|           5 | Clemmie    | Betchley   | 1973-11-07 | NULL         | 5 Spohn Circle         | Arlington        | TX    |   3675 |
|           6 | Elka       | Twiddell   | 1991-09-04 | 312-480-8498 | 7 Manley Drive         | Chicago          | IL    |   3073 |
|           7 | Ilene      | Dowson     | 1964-08-30 | 615-641-4759 | 50 Lillian Crossing    | Nashville        | TN    |   1672 |
|           8 | Thacher    | Naseby     | 1993-07-17 | 941-527-3977 | 538 Mosinee Center     | Sarasota         | FL    |    205 |
|           9 | Romola     | Rumgay     | 1992-05-23 | 559-181-3744 | 3520 Ohio Trail        | Visalia          | CA    |   1486 |
|          10 | Levy       | Mynett     | 1969-10-13 | 404-246-3370 | 68 Lawn Avenue         | Atlanta          | GA    |    796 |
+-------------+------------+------------+------------+--------------+------------------------+------------------+-------+--------+

-- 表order_items
+----------+------------+----------+------------+
| order_id | product_id | quantity | unit_price |
+----------+------------+----------+------------+
|        1 |          4 |        4 |       3.74 |
|        2 |          1 |        2 |       9.10 |
|        2 |          4 |        4 |       1.66 |
|        2 |          6 |        2 |       2.94 |
|        3 |          3 |       10 |       9.12 |
|        4 |          3 |        7 |       6.99 |
|        4 |         10 |        7 |       6.40 |
|        5 |          2 |        3 |       9.89 |
|        6 |          1 |        4 |       8.65 |
|        6 |          2 |        4 |       3.28 |
|        6 |          3 |        4 |       7.46 |
|        6 |          5 |        1 |       3.45 |
|        7 |          3 |        7 |       9.17 |
|        8 |          5 |        2 |       6.94 |
|        8 |          8 |        2 |       8.59 |
|        9 |          6 |        5 |       7.28 |
|       10 |          1 |       10 |       6.01 |
|       10 |          9 |        9 |       4.28 |
+----------+------------+----------+------------+

```

## MAX MIN AVG SUM COUNT
```sql
-- COUNT只针对非空值

SELECT MAX(invoice_total) AS highest, 
MIN(invoice_total) AS lowest, 
AVG(invoice_total) AS average,
SUM(invoice_total) AS total,
COUNT(DISTINCT client_id) AS number_of_invoices
FROM invoices
WHERE invoice_date > '2019-07-01';
+---------+--------+------------+---------+--------------------+
| highest | lowest | average    | total   | number_of_invoices |
+---------+--------+------------+---------+--------------------+
|  172.17 | 126.38 | 150.218571 | 1051.53 |                  3 |
+---------+--------+------------+---------+--------------------+


SELECT 'First half of 2019' AS date_range, 
    SUM(invoice_total) AS total_sales,
    SUM(payment_total) AS total_payments, 
    SUM(invoice_total - payment_total) AS what_we_expect 
    FROM invoices 
    WHERE invoice_date 
        BETWEEN '2019-01-01' AND '2019-06-30'
UNION
SELECT 'Second half of 2019' AS date_range, 
    SUM(invoice_total) AS total_sales, 
    SUM(payment_total) AS total_payments, 
    SUM(invoice_total - payment_total) AS what_we_expect
    FROM invoices 
    WHERE invoice_date 
        BETWEEN '2019-07-01' AND '2019-12-31'
UNION
SELECT 'Total' AS date_range, 
    SUM(invoice_total) AS total_sales, 
    SUM(payment_total) AS total_payments, 
    SUM(invoice_total - payment_total) AS what_we_expect 
    FROM invoices
    WHERE invoice_date 
        BETWEEN '2019-01-01' AND '2019-12-31';
+---------------------+-------------+----------------+----------------+
| date_range          | total_sales | total_payments | what_we_expect |
+---------------------+-------------+----------------+----------------+
| First half of 2019  |     1539.07 |         212.97 |        1326.10 |
| Second half of 2019 |     1051.53 |         148.41 |         903.12 |
| Total               |     2590.60 |         361.38 |        2229.22 |
+---------------------+-------------+----------------+----------------+

```

## GROUP BY
```sql
-- 查询每个日期对应每种付款方式的总收款量

SELECT p.date, pm.name, 
    SUM(p.amount) AS total_payments
FROM payments p JOIN payment_methods pm ON p.payment_method = pm.payment_method_id 
GROUP BY p.date, p.payment_method 
ORDER BY p.date;
+------------+-------------+----------------+
| date       | name        | total_payments |
+------------+-------------+----------------+
| 2019-01-03 | Credit Card |          74.55 |
| 2019-01-08 | Credit Card |          32.77 |
| 2019-01-08 | Cash        |          10.00 |
| 2019-01-11 | Credit Card |           0.03 |
| 2019-01-15 | Credit Card |         148.41 |
| 2019-01-26 | Credit Card |          87.44 |
| 2019-02-12 | Credit Card |           8.18 |
+------------+-------------+----------------+
```

## HAVING子句
```sql
-- 用在GROUP BY后面进行数据过滤
-- 使用orders、order_items、customers表，查找在'VA'州，并且总消费大于100的客户

-- 第一种写法
SELECT c.customer_id, c.first_name
FROM customers c
WHERE c.customer_id IN (
    SELECT o.customer_id
    FROM orders o
    JOIN order_items oi USING (order_id)
    GROUP BY customer_id
    HAVING SUM(oi.quantity * oi.unit_price) > 100
) AND c.state = 'VA';
+-------------+------------+
| customer_id | first_name |
+-------------+------------+
|           2 | Ines       |
+-------------+------------+

-- 第二种写法
SELECT c.customer_id, c.first_name, SUM(oi.quantity * oi.unit_price) AS total_spend
FROM customers c
JOIN orders o USING (customer_id)
JOIN order_items oi USING (order_id)
WHERE c.state = 'VA'
GROUP BY c.customer_id, c.first_name
HAVING total_spend > 100;
+-------------+------------+-------------+
| customer_id | first_name | total_spend |
+-------------+------------+-------------+
|           2 | Ines       |      157.92 |
+-------------+------------+-------------+
```

## ROLLUP
```sql
-- ROLLUP能统计分组的所有数据总和
-- 使用payments与payment_methods表计算每种付款方式的付款总和

SELECT pm.name, SUM(p.amount)
FROM payments p
JOIN payment_methods pm ON p.payment_method = pm.payment_method_id
GROUP BY pm.name WITH ROLLUP;
+-------------+---------------+
| name        | SUM(p.amount) |
+-------------+---------------+
| Cash        |         10.00 |
| Credit Card |        351.38 |
| NULL        |        361.38 |
+-------------+---------------+
```