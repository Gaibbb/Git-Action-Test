---
title: SQL 复杂查询函数
tags: []
id: '19'
categories:
  - - 'Linux'
date: 2023-08-26 15:30:00
---

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

-- 表employees
+-------------+------------+------------+-----------------------------+--------+------------+-----------+
| employee_id | first_name | last_name  | job_title                   | salary | reports_to | office_id |
+-------------+------------+------------+-----------------------------+--------+------------+-----------+
|       33391 | D'arcy     | Nortunen   | Account Executive           |  62871 |      37270 |         1 |
|       37270 | Yovonnda   | Magrannell | Executive Secretary         |  63996 |       NULL |        10 |
|       37851 | Sayer      | Matterson  | Statistician III            |  98926 |      37270 |         1 |
|       40448 | Mindy      | Crissil    | Staff Scientist             |  94860 |      37270 |         1 |
|       56274 | Keriann    | Alloisi    | VP Marketing                | 110150 |      37270 |         1 |
|       63196 | Alaster    | Scutchin   | Assistant Professor         |  32179 |      37270 |         2 |
|       67009 | North      | de Clerc   | VP Product Management       | 114257 |      37270 |         2 |
|       67370 | Elladine   | Rising     | Social Worker               |  96767 |      37270 |         2 |
|       68249 | Nisse      | Voysey     | Financial Advisor           |  52832 |      37270 |         2 |
|       72540 | Guthrey    | Iacopetti  | Office Assistant I          | 117690 |      37270 |         3 |
|       72913 | Kass       | Hefferan   | Computer Systems Analyst IV |  96401 |      37270 |         3 |
|       75900 | Virge      | Goodrum    | Information Systems Manager |  54578 |      37270 |         3 |
|       76196 | Mirilla    | Janowski   | Cost Accountant             | 119241 |      37270 |         3 |
|       80529 | Lynde      | Aronson    | Junior Executive            |  77182 |      37270 |         4 |
|       80679 | Mildrid    | Sokale     | Geologist II                |  67987 |      37270 |         4 |
|       84791 | Hazel      | Tarbert    | General Manager             |  93760 |      37270 |         4 |
|       95213 | Cole       | Kesterton  | Pharmacist                  |  86119 |      37270 |         4 |
|       96513 | Theresa    | Binney     | Food Chemist                |  47354 |      37270 |         5 |
|       98374 | Estrellita | Daleman    | Staff Accountant IV         |  70187 |      37270 |         5 |
|      115357 | Ivy        | Fearey     | Structural Engineer         |  92710 |      37270 |         5 |
+-------------+------------+------------+-----------------------------+--------+------------+-----------+

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

-- 表clients
+-----------+-------------+--------------------------+---------------+-------+--------------+
| client_id | name        | address                  | city          | state | phone        |
+-----------+-------------+--------------------------+---------------+-------+--------------+
|         1 | Vinte       | 3 Nevada Parkway         | Syracuse      | NY    | 315-252-7305 |
|         2 | Myworks     | 34267 Glendale Parkway   | Huntington    | WV    | 304-659-1170 |
|         3 | Yadel       | 096 Pawling Parkway      | San Francisco | CA    | 415-144-6037 |
|         4 | Kwideo      | 81674 Westerfield Circle | Waco          | TX    | 254-750-0784 |
|         5 | Topiclounge | 0863 Farmco Road         | Portland      | OR    | 971-888-9129 |
+-----------+-------------+--------------------------+---------------+-------+--------------+
```

## ALL
```sql
-- 下面两条语句等价
SELECT * 
FROM invoices 
WHERE invoice_total > (
    SELECT MAX(invoice_total) 
    FROM invoices 
    WHERE client_id = 3
);

SELECT * 
FROM invoices 
WHERE invoice_total > ALL (
    SELECT invoice_total 
    FROM invoices 
    WHERE client_id = 3
);
+------------+-------------+-----------+---------------+---------------+--------------+------------+--------------+
| invoice_id | number      | client_id | invoice_total | payment_total | invoice_date | due_date   | payment_date |
+------------+-------------+-----------+---------------+---------------+--------------+------------+--------------+
|          2 | 03-898-6735 |         5 |        175.32 |          8.18 | 2019-06-11   | 2019-07-01 | 2019-02-12   |
|          5 | 87-052-3121 |         5 |        169.36 |          0.00 | 2019-07-18   | 2019-08-07 | NULL         |
|          8 | 78-145-1093 |         1 |        189.12 |          0.00 | 2019-05-20   | 2019-06-09 | NULL         |
|          9 | 77-593-0081 |         5 |        172.17 |          0.00 | 2019-07-09   | 2019-07-29 | NULL         |
|         18 | 52-269-9803 |         5 |        180.17 |         42.77 | 2019-05-23   | 2019-06-12 | 2019-01-08   |
+------------+-------------+-----------+---------------+---------------+--------------+------------+--------------+
```

## ANY
```sql
-- 下面两条语句等价
SELECT *
FROM clients
WHERE client_id IN (
    SELECT client_id
    FROM invoices
    GROUP BY client_id
    HAVING COUNT(*) >= 2
    );

SELECT *
FROM clients
WHERE client_id = ANY (
    SELECT client_id
    FROM invoices
    GROUP BY client_id
    HAVING COUNT(*) >= 2
    );
+-----------+-------------+---------------------+---------------+-------+--------------+
| client_id | name        | address             | city          | state | phone        |
+-----------+-------------+---------------------+---------------+-------+--------------+
|         1 | Vinte       | 3 Nevada Parkway    | Syracuse      | NY    | 315-252-7305 |
|         3 | Yadel       | 096 Pawling Parkway | San Francisco | CA    | 415-144-6037 |
|         5 | Topiclounge | 0863 Farmco Road    | Portland      | OR    | 971-888-9129 |
+-----------+-------------+---------------------+---------------+-------+--------------+
```

## 相关子查询
```sql
-- 查询工资高于他办公室平均工资的员工

SELECT * 
FROM employees e 
WHERE salary > (
    SELECT AVG(salary) 
    FROM employees 
    WHERE office_id = e.office_id
    );
+-------------+------------+-----------+-----------------------+--------+------------+-----------+
| employee_id | first_name | last_name | job_title             | salary | reports_to | office_id |
+-------------+------------+-----------+-----------------------+--------+------------+-----------+
|       37851 | Sayer      | Matterson | Statistician III      |  98926 |      37270 |         1 |
|       40448 | Mindy      | Crissil   | Staff Scientist       |  94860 |      37270 |         1 |
|       56274 | Keriann    | Alloisi   | VP Marketing          | 110150 |      37270 |         1 |
|       67009 | North      | de Clerc  | VP Product Management | 114257 |      37270 |         2 |
|       67370 | Elladine   | Rising    | Social Worker         |  96767 |      37270 |         2 |
|       72540 | Guthrey    | Iacopetti | Office Assistant I    | 117690 |      37270 |         3 |
|       76196 | Mirilla    | Janowski  | Cost Accountant       | 119241 |      37270 |         3 |
|       84791 | Hazel      | Tarbert   | General Manager       |  93760 |      37270 |         4 |
|       95213 | Cole       | Kesterton | Pharmacist            |  86119 |      37270 |         4 |
|       98374 | Estrellita | Daleman   | Staff Accountant IV   |  70187 |      37270 |         5 |
|      115357 | Ivy        | Fearey    | Structural Engineer   |  92710 |      37270 |         5 |
+-------------+------------+-----------+-----------------------+--------+------------+-----------+
```

## EXISTS
```sql
-- 效率比IN高，无需生成一个表格供WHERE进行条件判断
SELECT *
FROM products
WHERE product_id NOT IN (
    SELECT product_id
    FROM order_items
);

SELECT * 
FROM products p 
WHERE not exists (
    SELECT * 
    FROM order_items 
    WHERE p.product_id = order_items.product_id
    );

+------------+-------------------+-------------------+------------+
| product_id | name              | quantity_in_stock | unit_price |
+------------+-------------------+-------------------+------------+
|          7 | Sweet Pea Sprouts |                98 |       3.29 |
+------------+-------------------+-------------------+------------+
```

## SELECT子查询
```sql
SELECT c.client_id, 
    c.name, 
    (SELECT SUM(invoice_total) FROM invoices WHERE c.client_id = client_id) AS total_sales,
    (SELECT AVG(invoice_total) FROM invoices) AS average,
    (SELECT total_sales average) AS difference
FROM clients c;
+-----------+-------------+-------------+------------+------------+
| client_id | name        | total_sales | average    | difference |
+-----------+-------------+-------------+------------+------------+
|         1 | Vinte       |      802.89 | 152.388235 | 650.501765 |
|         2 | Myworks     |      101.79 | 152.388235 | -50.598235 |
|         3 | Yadel       |      705.90 | 152.388235 | 553.511765 |
|         4 | Kwideo      |        NULL | 152.388235 |       NULL |
|         5 | Topiclounge |      980.02 | 152.388235 | 827.631765 |
+-----------+-------------+-------------+------------+------------+
```

## FROM子查询
```sql
-- 注意一定要别名
SELECT *
FROM (
    SELECT c.client_id, 
        c.name, 
        (SELECT SUM(invoice_total) FROM invoices WHERE c.client_id = client_id) AS total_sales,
        (SELECT AVG(invoice_total) FROM invoices) AS average,
        (SELECT total_sales - average) AS difference
    FROM clients c
) sales_summary
WHERE total_sales IS NOT NULL;
+-----------+-------------+-------------+------------+------------+
| client_id | name        | total_sales | average    | difference |
+-----------+-------------+-------------+------------+------------+
|         1 | Vinte       |      802.89 | 152.388235 | 152.388235 |
|         2 | Myworks     |      101.79 | 152.388235 | 152.388235 |
|         3 | Yadel       |      705.90 | 152.388235 | 152.388235 |
|         5 | Topiclounge |      980.02 | 152.388235 | 152.388235 |
+-----------+-------------+-------------+------------+------------+
```