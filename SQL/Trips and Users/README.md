Do give it an upvote and comment [here](https://leetcode.com/problems/trips-and-users/discuss/2387831/mysql-using-cte-only-no-joins)

## Using CTE 

```
# Write your MySQL query statement below
WITH cte
AS (
    SELECT 
        id,
        request_at AS Day,
        IF(status!='completed', 1, 0) AS Cancelled
    FROM Trips 
    WHERE client_id IN (SELECT users_id FROM Users WHERE banned='No')
        AND driver_id IN (SELECT users_id FROM Users WHERE banned='No')
        AND request_at BETWEEN '2013-10-01' AND '2013-10-03'
)

SELECT 
    Day, 
    ROUND(SUM(Cancelled) / COUNT(id), 2) AS 'Cancellation Rate'
FROM cte
GROUP BY Day
```

### 1. What's happening in CTE??
Without doing any counting of cancelled status, lets first filter what is asked:
* Request date should be between `2013-10-01` and `2013-10-03` 
	This is achieved using `WHERE request_at BETWEEN '2013-10-01' AND '2013-10-03'`
* Not including both drivers and clients that are banned, aka Banned='No' in Users table
			This is achieved using `WHERE client_id IN (SELECT users_id FROM Users WHERE banned='No')
        AND driver_id IN (SELECT users_id FROM Users WHERE banned='No')
Since both of these conditions need to be true, we use `AND` to use them together

Taking example of the example provided by Leetcode, then
#### The cte table will now look like this if I select request_at (Day), status, id
![image](https://assets.leetcode.com/users/images/a3131e65-2da4-49c1-9fe5-534680e58294_1659797558.7360885.jpeg)

Here, the red bracket represents (client_id, driver_id)

### 2. How to calculate the cancellation rate
Now for the calculation of cancellation rate, we HAVE to know the total no of cancelled trips both by drivers and clients each day. To do that, we simply represent the entry with cancellation status as `1` and we assign the other status i.e., 'completed' as `0`, so while summing over a day, these `0's` won't impact the total no of cancelled trips.
This is done by using `IF` clause, we can also use `CASE` clause here instead of `IF`
`IF(status!='completed', 1, 0) AS Cancelled`

#### Now, the cte table look like this, if we add the above column
![image](https://assets.leetcode.com/users/images/b73fa641-baf7-4d5d-83b3-080bff854fd7_1659797381.8994489.jpeg)
Now the cte table is ready, all we have to do is take a sum of 'Cancelled' column over each day i.e., `GROUP BY Day` and divide the sum of 'id' column to get the cancellation rate, then we round the rate using `ROUND` clause.
This is the code of it: 
```
SELECT 
    Day, 
    ROUND(SUM(Cancelled) / COUNT(id), 2) AS 'Cancellation Rate'
FROM cte
GROUP BY Day
```
**Why are we dividing by `COUNT(id)`?**
	When we perform a `COUNT` function over a day using `GROUP BY` and get the total no of unbanned trips that occured in that day. We do not have to calculated Completed trips here, because we already have `id` column to use.
##### Something like this
![image](https://assets.leetcode.com/users/images/f9f3c1a7-aa39-4915-a043-7571745e9f66_1659797628.0287173.jpeg)
We select just the required columns.

Further Readings:
1. [MySQL 8.0 Reference Manual: 13.2.15 WITH (Common Table Expressions)](https://dev.mysql.com/doc/refman/8.0/en/with.html)
2. [An Introduction to MySQL CTE](https://www.mysqltutorial.org/mysql-cte/)

Thanks for reading! 
If there's any error or doubt, do comment. Feedbacks are always appreciated!! Hopefully my handwriting is understandable.
