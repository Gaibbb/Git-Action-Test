---
title: SQL 函数
tags: []
id: '20'
categories:
  - - 'Linux'
date: 2023-08-26 18:31:00
---

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

-- 表products
+------------+------------------------------+-------------------+------------+
| product_id | name                         | quantity_in_stock | unit_price |
+------------+------------------------------+-------------------+------------+
|          1 | Foam Dinner Plate            |                70 |       1.21 |
|          2 | Pork - Bacon,back Peameal    |                49 |       4.65 |
|          3 | Lettuce - Romaine, Heart     |                38 |       3.35 |
|          4 | Brocolinni - Gaylan, Chinese |                90 |       4.53 |
|          5 | Sauce - Ranch Dressing       |                94 |       1.63 |
|          6 | Petit Baguette               |                14 |       2.39 |
|          7 | Sweet Pea Sprouts            |                98 |       3.29 |
|          8 | Island Oasis - Raspberry     |                26 |       0.74 |
|          9 | Longan                       |                67 |       2.26 |
|         10 | Broom - Push                 |                 6 |       1.09 |
+------------+------------------------------+-------------------+------------+

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
```

## 数值函数
```sql
-- ROUND用于对数字进行四舍五入，并可以选定精度
SELECT ROUND(5.7345, 2);
+------------------+
| ROUND(5.7345, 2) |
+------------------+
|             5.73 |
+------------------+

SELECT ROUND(5.7345);
+---------------+
| ROUND(5.7345) |
+---------------+
|             6 |
+---------------+

SELECT ROUND(5.7345, 1);
+------------------+
| ROUND(5.7345, 1) |
+------------------+
|              5.7 |
+------------------+

--  TRUNCATE用于截断数字
SELECT TRUNCATE(5.7345, 2);
+---------------------+
| TRUNCATE(5.7345, 2) |
+---------------------+
|                5.73 |
+---------------------+

-- CEILING返回大于或等于这个数字的最小整数
SELECT CEILING(5.7345);
+-----------------+
| CEILING(5.7345) |
+-----------------+
|               6 |
+-----------------+

-- FLOOR返回小于或等于这个数字的最小整数
SELECT FLOOR(5.7345);
+---------------+
| FLOOR(5.7345) |
+---------------+
|             5 |
+---------------+

-- ABS返回绝对值
SELECT ABS(-5.7345);
+--------------+
| ABS(-5.7345) |
+--------------+
|       5.7345 |
+--------------+

-- RAND返回0~1的随机数
SELECT RAND();
+----------------------+
| RAND()               |
+----------------------+
| 0.044234291691551256 |
+----------------------+
```

## 字符串函数
```sql
-- LENGTH用于返回字符串的长度
SELECT LENGTH('kano');
+----------------+
| LENGTH('kano') |
+----------------+
|              4 |
+----------------+

-- UPPER用于返回字符串的大写形式
SELECT UPPER('kano');
+---------------+
| UPPER('kano') |
+---------------+
| KANO          |
+---------------+

-- LOWER用于返回字符串的小写形式\
SELECT LOWER('KANO');
+---------------+
| LOWER('KANO') |
+---------------+
| kano          |
+---------------+

-- LTRIM用于去除前面带有空格的字符串
SELECT LTRIM('   kano');
+------------------+
| LTRIM('   kano') |
+------------------+
| kano             |
+------------------+

-- RTRIM用于去除后面带有空格的字符串
SELECT RTRIM('kano     ');
+--------------------+
| RTRIM('kano     ') |
+--------------------+
| kano               |
+--------------------+

-- TRIM用于去除两边带有空格的字符串
SELECT TRIM('     kano     ');
+------------------------+
| TRIM('     kano     ') |
+------------------------+
| kano                   |
+------------------------+

-- LEFT返回字符串左边的几个字符
SELECT LEFT('kano', 2);
+-----------------+
| LEFT('kano', 2) |
+-----------------+
| ka              |
+-----------------+

-- LEFT返回字符串右边的几个字符
SELECT RIGHT('kano', 2);
+------------------+
| RIGHT('kano', 2) |
+------------------+
| no               |
+------------------+

-- SUBSTRING用于返回任意位置的字符
SELECT SUBSTRING('kirisamekano', 9, 4);
+---------------------------------+
| SUBSTRING('kirisamekano', 9, 4) |
+---------------------------------+
| kano                            |
+---------------------------------+

-- LOCATE用于返回一个字符/字符串 在字符串中的位置
SELECT LOCATE('k','kirisamekano');
+----------------------------+
| LOCATE('k','kirisamekano') |
+----------------------------+
|                          1 |
+----------------------------+

SELECT LOCATE('kano','kirisamekano');
+-------------------------------+
| LOCATE('kano','kirisamekano') |
+-------------------------------+
|                             9 |
+-------------------------------+

-- 搜索不存在的字符时会返回0
SELECT LOCATE('C','kirisamekano');
+----------------------------+
| LOCATE('C','kirisamekano') |
+----------------------------+
|                          0 |
+----------------------------+

-- REPLACE用于替换一个字符或字符串
SELECT REPLACE('kirisamekano', 'kano', 'onak');
+-----------------------------------------+
| REPLACE('kirisamekano', 'kano', 'onak') |
+-----------------------------------------+
| kirisameonak                            |
+-----------------------------------------+

-- CONCAT用于将两个字符串拼接
SELECT CONCAT('kirisame', 'kano');
+----------------------------+
| CONCAT('kirisame', 'kano') |
+----------------------------+
| kirisamekano               |
+----------------------------+
```

## 日期函数
```sql
-- NOW获取当前电脑日期和时间
-- CURDATE获取当前电脑日期
-- CURTIME获取当前电脑时间
SELECT NOW(), CURDATE(), CURTIME();
+---------------------+------------+-----------+
| NOW()               | CURDATE()  | CURTIME() |
+---------------------+------------+-----------+
| 2023-08-26 17:13:05 | 2023-08-26 | 17:13:05  |
+---------------------+------------+-----------+

-- YEAR返回输入年份
-- MONTH返回输入月份
-- DAY返回输入日数
SELECT YEAR(NOW()), MONTH(NOW()), DAY(NOW());
+-------------+--------------+------------+
| YEAR(NOW()) | MONTH(NOW()) | DAY(NOW()) |
+-------------+--------------+------------+
|        2023 |            8 |         26 |
+-------------+--------------+------------+

-- HOUR返回当前小时
-- MINUTE返回当前分钟
-- SECOND返回当前秒数
SELECT HOUR(NOW()), MINUTE(NOW()), SECOND(NOW());
+-------------+---------------+---------------+
| HOUR(NOW()) | MINUTE(NOW()) | SECOND(NOW()) |
+-------------+---------------+---------------+
|          17 |            15 |            13 |
+-------------+---------------+---------------+

-- DAYNAME返回当前星期几
-- MONTHNAME返回当前月份
SELECT MONTHNAME(NOW()), DAYNAME(NOW());
+------------------+----------------+
| MONTHNAME(NOW()) | DAYNAME(NOW()) |
+------------------+----------------+
| August           | Saturday       |
+------------------+----------------+

-- EXTRACT返回想要获取的时间信息
SELECT EXTRACT(YEAR FROM NOW());
+--------------------------+
| EXTRACT(YEAR FROM NOW()) |
+--------------------------+
|                     2023 |
+--------------------------+

SELECT *
FROM orders
WHERE YEAR(order_date) >= 2019;
+----------+-------------+------------+--------+----------+--------------+------------+
| order_id | customer_id | order_date | status | comments | shipped_date | shipper_id |
+----------+-------------+------------+--------+----------+--------------+------------+
|        1 |           6 | 2019-01-30 |      1 | NULL     | NULL         |       NULL |
+----------+-------------+------------+--------+----------+--------------+------------+
```

## 日期格式化
```sql
-- DATE_FORMAT格式化日期输出
SELECT DATE_FORMAT(NOW(), '%M %d %Y');
+--------------------------------+
| DATE_FORMAT(NOW(), '%M %d %Y') |
+--------------------------------+
| August 26 2023                 |
+--------------------------------+

-- TIME_FORMAT格式化时间输出
SELECT TIME_FORMAT(NOW(), '%H:%i:%s');
+--------------------------------+
| TIME_FORMAT(NOW(), '%H:%i:%s') |
+--------------------------------+
| 17:19:35                       |
+--------------------------------+
```

## 日期计算
```sql
SELECT DATE_ADD(NOW(), INTERVAL 1 DAY);
+---------------------------------+
| DATE_ADD(NOW(), INTERVAL 1 DAY) |
+---------------------------------+
| 2023-08-27 17:21:31             |
+---------------------------------+

SELECT DATE_ADD(NOW(), INTERVAL -1 DAY);
+----------------------------------+
| DATE_ADD(NOW(), INTERVAL -1 DAY) |
+----------------------------------+
| 2023-08-25 17:21:48              |
+----------------------------------+

SELECT DATE_SUB(NOW(), INTERVAL 1 DAY);
+---------------------------------+
| DATE_SUB(NOW(), INTERVAL 1 DAY) |
+---------------------------------+
| 2023-08-25 17:22:02             |
+---------------------------------+

SELECT DATEDIFF('2023-06-04', NOW());
+-------------------------------+
| DATEDIFF('2023-06-04', NOW()) |
+-------------------------------+
|                           -83 |
+-------------------------------+

SELECT TIME_TO_SEC('09:00') - TIME_TO_SEC('09:02');
+---------------------------------------------+
| TIME_TO_SEC('09:00') - TIME_TO_SEC('09:02') |
+---------------------------------------------+
|                                        -120 |
+---------------------------------------------+
```

## IFNULL 与 COALESCE
```sql
SELECT order_id, IFNULL(shipper_id, 'Not assigned') AS shipper FROM orders;
+----------+--------------+
| order_id | shipper      |
+----------+--------------+
|        1 | Not assigned |
|        3 | Not assigned |
|        4 | Not assigned |
|        6 | Not assigned |
|        8 | Not assigned |
|        9 | 1            |
|       10 | 2            |
|        5 | 3            |
|        2 | 4            |
|        7 | 4            |
+----------+--------------+

SELECT order_id, COALESCE(shipper_id, comments,'Not assigned') AS shipper FROM orders;
+----------+-----------------------------------------------------------------------+
| order_id | shipper                                                               |
+----------+-----------------------------------------------------------------------+
|        1 | Not assigned                                                          |
|        2 | 4                                                                     |
|        3 | Not assigned                                                          |
|        4 | Not assigned                                                          |
|        5 | 3                                                                     |
|        6 | Aliquam erat volutpat. In congue.                                     |
|        7 | 4                                                                     |
|        8 | Mauris enim leo, rhoncus sed, vestibulum sit amet, cursus id, turpis. |
|        9 | 1                                                                     |
|       10 | 2                                                                     |
+----------+-----------------------------------------------------------------------+
```

## IF
```sql
SELECT p.product_id,
    p.name,
    COUNT(*) AS orders,
    IF(COUNT(*) > 1, 'Many times', 'Once') AS times
FROM products p
JOIN order_items USING (product_id)
GROUP BY p.product_id;
+------------+------------------------------+--------+------------+
| product_id | name                         | orders | times      |
+------------+------------------------------+--------+------------+
|          1 | Foam Dinner Plate            |      3 | Many times |
|          2 | Pork - Bacon,back Peameal    |      2 | Many times |
|          3 | Lettuce - Romaine, Heart     |      4 | Many times |
|          4 | Brocolinni - Gaylan, Chinese |      2 | Many times |
|          5 | Sauce - Ranch Dressing       |      2 | Many times |
|          6 | Petit Baguette               |      2 | Many times |
|          8 | Island Oasis - Raspberry     |      1 | Once       |
|          9 | Longan                       |      1 | Once       |
|         10 | Broom - Push                 |      1 | Once       |
+------------+------------------------------+--------+------------+
```

## CASE
```sql
-- 上面IF中例子的改写
SELECT p.product_id,
    p.name,
    (SELECT COUNT(order_id) FROM order_items WHERE product_id = p.product_id GROUP BY product_id) AS orders,
    CASE 
        WHEN (SELECT(orders)) > 1 THEN 'Many times' 
        WHEN (SELECT(orders)) = 1 THEN 'Once'
        ELSE 'Never'
    END AS times
FROM products p;
+------------+------------------------------+--------+------------+
| product_id | name                         | orders | times      |
+------------+------------------------------+--------+------------+
|          1 | Foam Dinner Plate            |      3 | Many times |
|          2 | Pork - Bacon,back Peameal    |      2 | Many times |
|          3 | Lettuce - Romaine, Heart     |      4 | Many times |
|          4 | Brocolinni - Gaylan, Chinese |      2 | Many times |
|          5 | Sauce - Ranch Dressing       |      2 | Many times |
|          6 | Petit Baguette               |      2 | Many times |
|          7 | Sweet Pea Sprouts            |   NULL | Never      |
|          8 | Island Oasis - Raspberry     |      1 | Once       |
|          9 | Longan                       |      1 | Once       |
|         10 | Broom - Push                 |      1 | Once       |
+------------+------------------------------+--------+------------+

-- 前面SQL中的最后一部分"UNION 联合"改写
SELECT CONCAT(first_name, ' ' , last_name) AS customer, 
    points,
    CASE 
        WHEN points < 2000 THEN 'Bronze' 
        WHEN points >= 2000 THEN 'Silver' 
        WHEN points > 3000 THEN 'Glod' 
    END AS category 
FROM customers
ORDER BY points DESC;
+-------------------+--------+----------+
| customer          | points | category |
+-------------------+--------+----------+
| Clemmie Betchley  |   3675 | Glod     |
| Elka Twiddell     |   3073 | Glod     |
| Freddi Boagey     |   2967 | Silver   |
| Babara MacCaffrey |   2273 | Silver   |
| Ilene Dowson      |   1672 | Bronze   |
| Romola Rumgay     |   1486 | Bronze   |
| Ines Brushfield   |    947 | Bronze   |
| Levy Mynett       |    796 | Bronze   |
| Ambur Roseburgh   |    457 | Bronze   |
| Thacher Naseby    |    205 | Bronze   |
+-------------------+--------+----------+
```