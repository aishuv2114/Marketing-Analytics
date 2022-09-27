## Marketing Analytics on E-commerce dataset using SQL - Part 4 of 5

### **User Analysis**

**Objective**
Analyzing repeat visits helps you understand user behavior and identify some of your most valuable customers

1)The marketing director has a request to pull data on how many of the website visitors come back for another session for 2014 to date.

```sql
WITH cust_2014
     AS (SELECT user_id
         FROM   website_sessions
         WHERE  ( Date(created_at) >= '2014-01-01'
                  AND Date(created_at) < '2014-11-01' )
                AND is_repeat_session = 0),
     sessions
     AS (SELECT user_id,
                Sum(is_repeat_session) AS 'New_or_Repeat'
         FROM   website_sessions
         WHERE  ( Date(created_at) >= '2014-01-01'
                  AND Date(created_at) < '2014-11-01' )
                AND user_id IN (SELECT user_id
                                FROM   cust_2014)
         GROUP  BY user_id)
SELECT new_or_repeat,
       Count(DISTINCT user_id) AS '#Users'
FROM   sessions
GROUP  BY new_or_repeat;   

        
```

**Output**  
![Screenshot_26](https://user-images.githubusercontent.com/113862057/192419258-d4f2fe17-3c1a-4ef6-a5b2-1f357cb4856d.png)

**Insights**  
- A fair number of our customers do come back to the site after the first session.


2)The marketing director has another request to understand the minimum, maximum, and average time between the first and second session for
customers who do come back for the same time period.

```sql
WITH cust_2014
     AS (SELECT user_id
         FROM   website_sessions
         WHERE  ( Date(created_at) >= '2014-01-01'
                  AND Date(created_at) < '2014-11-01' )
                AND is_repeat_session = 0),
     userorders_rank
     AS (SELECT user_id,
                created_at,
                Rank()
                  OVER (
                    partition BY user_id
                    ORDER BY created_at) AS 'Order_Rank'
         FROM   website_sessions
         WHERE  user_id IN (SELECT user_id
                            FROM   cust_2014)),
     sessions
     AS (SELECT user_id,
                order_rank,
                Date(created_at)               AS created_at,
                Date(Lag(created_at, 1)
                       OVER (
                         partition BY user_id
                         ORDER BY created_at)) AS 'Previous'
         FROM   userorders_rank),
     difference_btw_sessions
     AS (SELECT user_id,
                created_at                     AS 'Second Session',
                previous                       AS 'First Session',
                Datediff(created_at, previous) AS Diff_between_First_Second
         FROM   sessions
         WHERE  order_rank = 2)
SELECT Min(diff_between_first_second)                           AS
       min_days_first_to_second,
       Max(diff_between_first_second)                           AS
       max_days_first_to_second,
       Sum(diff_between_first_second) / Count(DISTINCT user_id) AS
       'Avg_days_first_to_second'
FROM   difference_btw_sessions; 
```


**Output**  
![Screenshot_27](https://user-images.githubusercontent.com/113862057/192419622-94b2fdcd-6471-4578-ad29-a2741e2ba262.png)

**Insights**  
The repeat visitors are coming back about a month later, on average.

3)The marketing director is also curious to understand the channels they come back through,  if it’s all direct type in, or paid search ads multiple times.

```sql
SELECT CASE
         WHEN utm_campaign IS NULL
              AND ( http_referer LIKE '%gsearch%'
                     OR http_referer LIKE '%bsearch%' ) THEN 'Organic_Search'
         WHEN utm_campaign = 'nonbrand' THEN 'paid_nonbrand'
         WHEN utm_campaign = 'brand' THEN 'paid_brand'
         WHEN utm_source IS NULL
              AND http_referer IS NULL THEN 'Direct_Type_In'
         WHEN utm_source = 'socialbook' THEN 'paid_social'
         ELSE NULL
       END        AS channel_group,
       Count(CASE
               WHEN is_repeat_session = 0 THEN website_session_id
             END) AS New_Customers,
       Count(CASE
               WHEN is_repeat_session = 1 THEN website_session_id
             END) AS Rpeat_Customers
FROM   website_sessions
WHERE  Date(created_at) >= '2014-01-01'
       AND Date(created_at) <= '2014-11-04'
GROUP  BY channel_group; 
```

**Output**  
![Screenshot_28](https://user-images.githubusercontent.com/113862057/192420570-d4c90a45-279a-44f1-b428-4bcdd48f097c.png)

**Insights**  
- The customers come back for repeat visits,they come mainly through organic search, direct type in,and paid brand.
- Only about 1/3 come through a paid channel, and brand clicks are cheaper than nonbrand. 


4)


