---
title: SQL JOIN
tags: []
id: '16'
categories:
  - - 'Linux'
date: 2023-08-26 9:41:00
---

# 连接 JOIN

## 表
```sql
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

-- 表shipper
+------------+-----------------------------+
| shipper_id | name                        |
+------------+-----------------------------+
|          1 | Hettinger LLC               |
|          2 | Schinner-Predovic           |
|          3 | Satterfield LLC             |
|          4 | Mraz, Renner and Nolan      |
|          5 | Waters, Mayert and Prohaska |
+------------+-----------------------------+

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

-- 表order_item_notes
+---------+----------+------------+-------------+
| note_id | order_Id | product_id | note        |
+---------+----------+------------+-------------+
|       1 |        1 |          2 | first note  |
|       2 |        1 |          2 | second note |
+---------+----------+------------+-------------+
```

## CROSS JOIN
```sql
-- CROSS JOIN 把两个表的数据一一对应
-- 比如左表是颜色 右表是型号
-- 使用该语句就会使每种颜色对应每一个型号都出现一次
SELECT *
FROM orders
CROSS JOIN shippers;

+----------+-------------+------------+------------+------------+-----------------------------+
| order_id | customer_id | order_date | shipper_id | shipper_id | name                        |
+----------+-------------+------------+------------+------------+-----------------------------+
|        1 |           6 | 2019-01-30 |       NULL |          5 | Waters, Mayert and Prohaska |
|        1 |           6 | 2019-01-30 |       NULL |          4 | Mraz, Renner and Nolan      |
|        1 |           6 | 2019-01-30 |       NULL |          3 | Satterfield LLC             |
|        1 |           6 | 2019-01-30 |       NULL |          2 | Schinner-Predovic           |
|        1 |           6 | 2019-01-30 |       NULL |          1 | Hettinger LLC               |
|        2 |           7 | 2018-08-02 |          4 |          5 | Waters, Mayert and Prohaska |
|        2 |           7 | 2018-08-02 |          4 |          4 | Mraz, Renner and Nolan      |
|        2 |           7 | 2018-08-02 |          4 |          3 | Satterfield LLC             |
|        2 |           7 | 2018-08-02 |          4 |          2 | Schinner-Predovic           |
|        2 |           7 | 2018-08-02 |          4 |          1 | Hettinger LLC               |
|        3 |           8 | 2017-12-01 |       NULL |          5 | Waters, Mayert and Prohaska |
|        3 |           8 | 2017-12-01 |       NULL |          4 | Mraz, Renner and Nolan      |
|        3 |           8 | 2017-12-01 |       NULL |          3 | Satterfield LLC             |
|        3 |           8 | 2017-12-01 |       NULL |          2 | Schinner-Predovic           |
|        3 |           8 | 2017-12-01 |       NULL |          1 | Hettinger LLC               |
|        4 |           2 | 2017-01-22 |       NULL |          5 | Waters, Mayert and Prohaska |
|        4 |           2 | 2017-01-22 |       NULL |          4 | Mraz, Renner and Nolan      |
|        4 |           2 | 2017-01-22 |       NULL |          3 | Satterfield LLC             |
|        4 |           2 | 2017-01-22 |       NULL |          2 | Schinner-Predovic           |
|        4 |           2 | 2017-01-22 |       NULL |          1 | Hettinger LLC               |
|        5 |           5 | 2017-08-25 |          3 |          5 | Waters, Mayert and Prohaska |
|        5 |           5 | 2017-08-25 |          3 |          4 | Mraz, Renner and Nolan      |
|        5 |           5 | 2017-08-25 |          3 |          3 | Satterfield LLC             |
|        5 |           5 | 2017-08-25 |          3 |          2 | Schinner-Predovic           |
|        5 |           5 | 2017-08-25 |          3 |          1 | Hettinger LLC               |
|        6 |          10 | 2018-11-18 |       NULL |          5 | Waters, Mayert and Prohaska |
|        6 |          10 | 2018-11-18 |       NULL |          4 | Mraz, Renner and Nolan      |
|        6 |          10 | 2018-11-18 |       NULL |          3 | Satterfield LLC             |
|        6 |          10 | 2018-11-18 |       NULL |          2 | Schinner-Predovic           |
|        6 |          10 | 2018-11-18 |       NULL |          1 | Hettinger LLC               |
|        7 |           2 | 2018-09-22 |          4 |          5 | Waters, Mayert and Prohaska |
|        7 |           2 | 2018-09-22 |          4 |          4 | Mraz, Renner and Nolan      |
|        7 |           2 | 2018-09-22 |          4 |          3 | Satterfield LLC             |
|        7 |           2 | 2018-09-22 |          4 |          2 | Schinner-Predovic           |
|        7 |           2 | 2018-09-22 |          4 |          1 | Hettinger LLC               |
|        8 |           5 | 2018-06-08 |       NULL |          5 | Waters, Mayert and Prohaska |
|        8 |           5 | 2018-06-08 |       NULL |          4 | Mraz, Renner and Nolan      |
|        8 |           5 | 2018-06-08 |       NULL |          3 | Satterfield LLC             |
|        8 |           5 | 2018-06-08 |       NULL |          2 | Schinner-Predovic           |
|        8 |           5 | 2018-06-08 |       NULL |          1 | Hettinger LLC               |
|        9 |          10 | 2017-07-05 |          1 |          5 | Waters, Mayert and Prohaska |
|        9 |          10 | 2017-07-05 |          1 |          4 | Mraz, Renner and Nolan      |
|        9 |          10 | 2017-07-05 |          1 |          3 | Satterfield LLC             |
|        9 |          10 | 2017-07-05 |          1 |          2 | Schinner-Predovic           |
|        9 |          10 | 2017-07-05 |          1 |          1 | Hettinger LLC               |
|       10 |           6 | 2018-04-22 |          2 |          5 | Waters, Mayert and Prohaska |
|       10 |           6 | 2018-04-22 |          2 |          4 | Mraz, Renner and Nolan      |
|       10 |           6 | 2018-04-22 |          2 |          3 | Satterfield LLC             |
|       10 |           6 | 2018-04-22 |          2 |          2 | Schinner-Predovic           |
|       10 |           6 | 2018-04-22 |          2 |          1 | Hettinger LLC               |
+----------+-------------+------------+------------+------------+-----------------------------+
```

## INNER JOIN
```sql
-- INNER JOIN是默认的连接
-- 显式连接
SELECT *
FROM orders
JOIN shippers ON orders.shipper_id = shippers.shipper_id;
+----------+-------------+------------+------------+------------+------------------------+
| order_id | customer_id | order_date | shipper_id | shipper_id | name                   |
+----------+-------------+------------+------------+------------+------------------------+
|        2 |           7 | 2018-08-02 |          4 |          4 | Mraz, Renner and Nolan |
|        5 |           5 | 2017-08-25 |          3 |          3 | Satterfield LLC        |
|        7 |           2 | 2018-09-22 |          4 |          4 | Mraz, Renner and Nolan |
|        9 |          10 | 2017-07-05 |          1 |          1 | Hettinger LLC          |
|       10 |           6 | 2018-04-22 |          2 |          2 | Schinner-Predovic      |
+----------+-------------+------------+------------+------------+------------------------+

-- 隐式连接
SELECT *
FROM orders, shippers
WHERE orders.shipper_id = shippers.shipper_id;
-- 结果与上面的一样

-- 多表连接
SELECT c.first_name, o.order_id, o.order_date, s.name
FROM orders o 
JOIN shippers s ON s.shipper_id = o.shipper_id
JOIN customers c ON c.customer_id = o.customer_id;
+------------+----------+------------+------------------------+
| first_name | order_id | order_date | name                   |
+------------+----------+------------+------------------------+
| Ilene      |        2 | 2018-08-02 | Mraz, Renner and Nolan |
| Clemmie    |        5 | 2017-08-25 | Satterfield LLC        |
| Ines       |        7 | 2018-09-22 | Mraz, Renner and Nolan |
| Levy       |        9 | 2017-07-05 | Hettinger LLC          |
| Elka       |       10 | 2018-04-22 | Schinner-Predovic      |
+------------+----------+------------+------------------------+

-- 复合连接条件
SELECT *
FROM order_items oi
JOIN order_item_notes oin
ON oi.order_id = oin.order_id
AND oi.product_id = oin.product_id;

Empty set (0.01 sec)
-- 很明显使没有结果的
```

## OUTER JOIN
在 SQL 中，OUTER JOIN 是一种关联查询方式，它根据指定的关联条件，将两个表中满足条件的行组合在一起，并 包含没有匹配的行 。

在 OUTER JOIN 中，包括 LEFT OUTER JOIN 和 RIGHT OUTER JOIN 两种类型，它们分别表示查询左表和右表的所有行（即使没有被匹配），再加上满足条件的交集部分。

```sql
-- 左连接
SELECT c.customer_id, c.first_name, c.state, c.points, o.order_id
FROM customers c
LEFT JOIN orders o
ON o.customer_id = c.customer_id;
+-------------+------------+-------+--------+----------+
| customer_id | first_name | state | points | order_id |
+-------------+------------+-------+--------+----------+
|           1 | Babara     | MA    |   2273 |     NULL |
|           2 | Ines       | VA    |    947 |        4 |
|           2 | Ines       | VA    |    947 |        7 |
|           3 | Freddi     | CO    |   2967 |     NULL |
|           4 | Ambur      | FL    |    457 |     NULL |
|           5 | Clemmie    | TX    |   3675 |        5 |
|           5 | Clemmie    | TX    |   3675 |        8 |
|           6 | Elka       | IL    |   3073 |        1 |
|           6 | Elka       | IL    |   3073 |       10 |
|           7 | Ilene      | TN    |   1672 |        2 |
|           8 | Thacher    | FL    |    205 |        3 |
|           9 | Romola     | CA    |   1486 |     NULL |
|          10 | Levy       | GA    |    796 |        6 |
|          10 | Levy       | GA    |    796 |        9 |
+-------------+------------+-------+--------+----------+

-- 右连接
SELECT c.customer_id, c.first_name, c.state, c.points, o.order_id
FROM customers c
RIGHT JOIN orders o
ON o.customer_id = c.customer_id;
+-------------+------------+-------+--------+----------+
| customer_id | first_name | state | points | order_id |
+-------------+------------+-------+--------+----------+
|           2 | Ines       | VA    |    947 |        4 |
|           2 | Ines       | VA    |    947 |        7 |
|           5 | Clemmie    | TX    |   3675 |        5 |
|           5 | Clemmie    | TX    |   3675 |        8 |
|           6 | Elka       | IL    |   3073 |        1 |
|           6 | Elka       | IL    |   3073 |       10 |
|           7 | Ilene      | TN    |   1672 |        2 |
|           8 | Thacher    | FL    |    205 |        3 |
|          10 | Levy       | GA    |    796 |        6 |
|          10 | Levy       | GA    |    796 |        9 |
+-------------+------------+-------+--------+----------+

-- 以下两条SQL语句返回结果相同
SELECT c.customer_id, c.first_name, c.state, c.points, o.order_id
FROM customers c
LEFT JOIN orders o
ON o.customer_id = c.customer_id;

SELECT c.customer_id, c.first_name, c.state, c.points, o.order_id 
FROM orders o 
RIGHT JOIN customers c 
ON o.customer_id = c.customer_id;
+-------------+------------+-------+--------+----------+
| customer_id | first_name | state | points | order_id |
+-------------+------------+-------+--------+----------+
|           1 | Babara     | MA    |   2273 |     NULL |
|           2 | Ines       | VA    |    947 |        4 |
|           2 | Ines       | VA    |    947 |        7 |
|           3 | Freddi     | CO    |   2967 |     NULL |
|           4 | Ambur      | FL    |    457 |     NULL |
|           5 | Clemmie    | TX    |   3675 |        5 |
|           5 | Clemmie    | TX    |   3675 |        8 |
|           6 | Elka       | IL    |   3073 |        1 |
|           6 | Elka       | IL    |   3073 |       10 |
|           7 | Ilene      | TN    |   1672 |        2 |
|           8 | Thacher    | FL    |    205 |        3 |
|           9 | Romola     | CA    |   1486 |     NULL |
|          10 | Levy       | GA    |    796 |        6 |
|          10 | Levy       | GA    |    796 |        9 |
+-------------+------------+-------+--------+----------+

-- 多表连接 和上方的多表内连接是一样的
SELECT c.first_name, o.order_id, o.order_date, s.name
FROM orders o 
LEFT JOIN shippers s ON s.shipper_id = o.shipper_id
JOIN customers c ON c.customer_id = o.customer_id;
+------------+----------+------------+------------------------+
| first_name | order_id | order_date | name                   |
+------------+----------+------------+------------------------+
| Elka       |        1 | 2019-01-30 | NULL                   |
| Ilene      |        2 | 2018-08-02 | Mraz, Renner and Nolan |
| Thacher    |        3 | 2017-12-01 | NULL                   |
| Ines       |        4 | 2017-01-22 | NULL                   |
| Clemmie    |        5 | 2017-08-25 | Satterfield LLC        |
| Levy       |        6 | 2018-11-18 | NULL                   |
| Ines       |        7 | 2018-09-22 | Mraz, Renner and Nolan |
| Clemmie    |        8 | 2018-06-08 | NULL                   |
| Levy       |        9 | 2017-07-05 | Hettinger LLC          |
| Elka       |       10 | 2018-04-22 | Schinner-Predovic      |
+------------+----------+------------+------------------------+
```

## USING子句
```sql
-- USING子句用来简化连接的书写
-- 在两个表中都拥有相同的字段名的时候使用
SELECT shipper_id, order_id, customer_id, order_date, shipped_date, name
FROM orders
JOIN shippers
USING(shipper_id);

-- 等价于
SELECT *
FROM orders
JOIN shippers ON orders.shipper_id = shippers.shipper_id;
+------------+----------+-------------+------------+--------------+------------------------+
| shipper_id | order_id | customer_id | order_date | shipped_date | name                   |
+------------+----------+-------------+------------+--------------+------------------------+
|          1 |        9 |          10 | 2017-07-05 | 2017-07-06   | Hettinger LLC          |
|          2 |       10 |           6 | 2018-04-22 | 2018-04-23   | Schinner-Predovic      |
|          3 |        5 |           5 | 2017-08-25 | 2017-08-26   | Satterfield LLC        |
|          4 |        2 |           7 | 2018-08-02 | 2018-08-03   | Mraz, Renner and Nolan |
|          4 |        7 |           2 | 2018-09-22 | 2018-09-23   | Mraz, Renner and Nolan |
+------------+----------+-------------+------------+--------------+------------------------+

-- 同样对外连接也适用
SELECT c.customer_id, c.first_name, c.state, c.points, o.order_id
FROM customers c
LEFT JOIN orders o
USING (customer_id);

-- 等价于
SELECT c.customer_id, c.first_name, c.state, c.points, o.order_id
FROM customers c
LEFT JOIN orders o
ON o.customer_id = c.customer_id;
+-------------+------------+-------+--------+----------+
| customer_id | first_name | state | points | order_id |
+-------------+------------+-------+--------+----------+
|           1 | Babara     | MA    |   2273 |     NULL |
|           2 | Ines       | VA    |    947 |        4 |
|           2 | Ines       | VA    |    947 |        7 |
|           3 | Freddi     | CO    |   2967 |     NULL |
|           4 | Ambur      | FL    |    457 |     NULL |
|           5 | Clemmie    | TX    |   3675 |        5 |
|           5 | Clemmie    | TX    |   3675 |        8 |
|           6 | Elka       | IL    |   3073 |        1 |
|           6 | Elka       | IL    |   3073 |       10 |
|           7 | Ilene      | TN    |   1672 |        2 |
|           8 | Thacher    | FL    |    205 |        3 |
|           9 | Romola     | CA    |   1486 |     NULL |
|          10 | Levy       | GA    |    796 |        6 |
|          10 | Levy       | GA    |    796 |        9 |
+-------------+------------+-------+--------+----------+
```

## NATURAL JOIN 自然连接
```sql
-- 会让数据库自己判断如何连接，不可控性太大，不建议使用
SELECT shipper_id, order_id, customer_id, order_date, shipped_date, name
FROM orders
NATURAL JOIN shippers;
+------------+----------+-------------+------------+--------------+------------------------+
| shipper_id | order_id | customer_id | order_date | shipped_date | name                   |
+------------+----------+-------------+------------+--------------+------------------------+
|          1 |        9 |          10 | 2017-07-05 | 2017-07-06   | Hettinger LLC          |
|          2 |       10 |           6 | 2018-04-22 | 2018-04-23   | Schinner-Predovic      |
|          3 |        5 |           5 | 2017-08-25 | 2017-08-26   | Satterfield LLC        |
|          4 |        2 |           7 | 2018-08-02 | 2018-08-03   | Mraz, Renner and Nolan |
|          4 |        7 |           2 | 2018-09-22 | 2018-09-23   | Mraz, Renner and Nolan |
+------------+----------+-------------+------------+--------------+------------------------+
```

## UNION 联合
```sql
-- 用于对多个查询结果进行拼贴
SELECT name
FROM shippers
UNION
SELECT first_name
FROM customers;
+-----------------------------+
| name                        |
+-----------------------------+
| Hettinger LLC               |
| Schinner-Predovic           |
| Satterfield LLC             |
| Mraz, Renner and Nolan      |
| Waters, Mayert and Prohaska |
| Babara                      |
| Ines                        |
| Freddi                      |
| Ambur                       |
| Clemmie                     |
| Elka                        |
| Ilene                       |
| Thacher                     |
| Romola                      |
| Levy                        |
+-----------------------------+

-- 交换顺序会使得字段名发生改变
SELECT first_name
FROM customers
UNION
SELECT name
FROM shippers;
+-----------------------------+
| first_name                  |
+-----------------------------+
| Babara                      |
| Ines                        |
| Freddi                      |
| Ambur                       |
| Clemmie                     |
| Elka                        |
| Ilene                       |
| Thacher                     |
| Romola                      |
| Levy                        |
| Hettinger LLC               |
| Schinner-Predovic           |
| Satterfield LLC             |
| Mraz, Renner and Nolan      |
| Waters, Mayert and Prohaska |
+-----------------------------+

-- 具体应用
SELECT customer_id, first_name, points, 'Bronze' AS type
FROM customers
WHERE points < 2000
UNION
SELECT customer_id, first_name, points, 'Silver' AS type
FROM customers
WHERE points BETWEEN 2000 AND 3000
UNION
SELECT customer_id, first_name, points, 'Gold' AS type
FROM customers
WHERE points > 3000
ORDER BY first_name;
+-------------+------------+--------+--------+
| customer_id | first_name | points | type   |
+-------------+------------+--------+--------+
|           4 | Ambur      |    457 | Bronze |
|           1 | Babara     |   2273 | Silver |
|           5 | Clemmie    |   3675 | Gold   |
|           6 | Elka       |   3073 | Gold   |
|           3 | Freddi     |   2967 | Silver |
|           7 | Ilene      |   1672 | Bronze |
|           2 | Ines       |    947 | Bronze |
|          10 | Levy       |    796 | Bronze |
|           9 | Romola     |   1486 | Bronze |
|           8 | Thacher    |    205 | Bronze |
+-------------+------------+--------+--------+
```