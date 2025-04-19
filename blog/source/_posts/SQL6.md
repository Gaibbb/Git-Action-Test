---
title: SQL VIEW
tags: []
id: '21'
categories:
  - - 'Linux'
date: 2023-08-26 19:22:00
---

## 表
```sql
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
```

## CREATE VIEW
```sql
CREATE VIEW clients_balance AS
SELECT c.client_id,
    c.name,
    SUM(i.invoice_total - i.payment_total) AS balance
FROM clients c
JOIN invoices i USING (client_id)
GROUP BY c.client_id, c.name;

SELECT * FROM clients_balance;
+-----------+-------------+---------+
| client_id | name        | balance |
+-----------+-------------+---------+
|         2 | Myworks     |  101.79 |
|         5 | Topiclounge |  841.63 |
|         3 | Yadel       |  557.46 |
|         1 | Vinte       |  728.34 |
+-----------+-------------+---------+
```

## 更新或修改视图 (可更新视图)
```sql
-- 第一种 直接删除重来
DROP VIEW clients_balance;

-- 第二种 REPLACE
CREATE OR REPLACE VIEW clients_balance AS
SELECT c.client_id,
    c.name,
    c.state,
    SUM(i.invoice_total - i.payment_total) AS balance
FROM clients c
JOIN invoices i USING (client_id)
GROUP BY c.client_id, c.name;
+-----------+-------------+-------+---------+
| client_id | name        | state | balance |
+-----------+-------------+-------+---------+
|         2 | Myworks     | WV    |  101.79 |
|         5 | Topiclounge | OR    |  841.63 |
|         3 | Yadel       | CA    |  557.46 |
|         1 | Vinte       | NY    |  728.34 |
+-----------+-------------+-------+---------+

-- 更新的话和表的更新差不多
UPDATE View SET ...;
DELETE FROM View ....;
INSERT View INTO ...;

-- 用一个新的视图解释
CREATE VIEW clients_no_balance AS
SELECT c.client_id,
    c.name,
    c.state
FROM clients c;

--更新一下
UPDATE clients_no_balance
SET state = 'VA'
WHERE client_id = 1;
+-----------+-------------+-------+
| client_id | name        | state |
+-----------+-------------+-------+
|         1 | Vinte       | VA    |
|         2 | Myworks     | WV    |
|         3 | Yadel       | CA    |
|         4 | Kwideo      | TX    |
|         5 | Topiclounge | OR    |
+-----------+-------------+-------+
```

## 不可更新视图
```sql
--当视图中使用了以下函数时，视图不可更新
DISTINCT
MIN, MAX, SUM, AVG, etc...
GROUP BY / HAVING
UNION

-- 当尝试修改clients_balance(使用了SUM)
UPDATE clients_balance 
SET state = 'VA'
WHERE client_id = 2;
ERROR 1288 (HY000): The target table clients_balance of the UPDATE is not updatable
```

## WITH CHECK OPTION
```sql
-- 在通过UPDATE视图的数据时，有可能会导致行的删除
-- 通过创建视图时加上WITH CHECK OPTION可以防止此类事件发生
CREATE OR REPLACE VIEW clients_no_balance AS
SELECT c.client_id,
    c.name,
    c.state
FROM clients c
CHECK WITH OPTION;
```