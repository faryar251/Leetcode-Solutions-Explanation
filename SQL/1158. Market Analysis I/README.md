### 1. Using just LEFT JOIN (The simplest and easy to follow solution)
```
SELECT 
    u.user_id AS buyer_id,
    u.join_date,
    COUNT(o.order_date) AS orders_in_2019
FROM Users u
LEFT JOIN Orders o
ON
    o.buyer_id = u.user_id AND YEAR(o.order_date) = 2019
GROUP BY u.user_id
```
Here, we are filtering the elements we want to join with Users table i.e., orders made in 2019 are only allowed using `AND` while joining the tables together. Then we count it. `COUNT` doesn't include the null values. This is why we don't need to use `IFNULL`

##### A variation of the above query, using `SUM` with `CASE`
```
SELECT 
    u.user_id AS buyer_id,
    u.join_date,
    SUM(CASE WHEN YEAR(o.order_date)=2019 THEN 1 ELSE 0 END) AS orders_in_2019
FROM Users u
LEFT JOIN Orders o
ON 
    o.buyer_id = u.user_id
GROUP BY u.user_id
```

Note: We have also removed the `AND `condition from `LEFT JOIN` which was filtering the 'ordering in 2019' condition while joining the Users table.

##### 2nd variation of the above query, using `SUM` with `IF`
```
SELECT 
    u.user_id AS buyer_id,
    u.join_date,
    SUM(IF(YEAR(o.order_date)=2019, 1, 0)) AS orders_in_2019
FROM Users u
LEFT JOIN Orders o
ON 
    o.buyer_id = u.user_id
GROUP BY u.user_id

```


### 2. Using Subquery
```
SELECT 
    u.user_id AS buyer_id,
    u.join_date,
    IFNULL(orders_in_2019, 0) AS orders_in_2019
FROM Users u
LEFT JOIN (SELECT buyer_id, COUNT(order_date) AS orders_in_2019
      FROM Orders
      WHERE YEAR(order_date) = 2019
      GROUP BY buyer_id) o
ON u.user_id = o.buyer_id
```

### 3. Using WITH CTE (Common Table Expressions)
```
WITH cte 
AS (
    SELECT 
        o.buyer_id, 
        COUNT(o.order_date) AS orders_in_2019
    FROM Orders o
    WHERE YEAR(o.order_date) = 2019
    GROUP BY o.buyer_id)
    
SELECT 
    u.user_id AS buyer_id,
    u.join_date,
    IFNULL(orders_in_2019, 0) AS orders_in_2019
FROM Users u
LEFT JOIN cte c ON u.user_id = c.buyer_id
```

##### Note:
`YEAR(o.order_date) = 2019` can be replaced by  `o.order_date LIKE '2019%'` OR  `o.order_date BETWEEN '2019-01-01 00:00:00' AND '2019-12-31 23:59:59'`

Thanks for reading! 
If there's any error or doubt, do comment. Feedbacks are always appreciated!!
