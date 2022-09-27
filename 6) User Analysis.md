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
![Screenshot_21](https://user-images.githubusercontent.com/113862057/192411215-65ab581a-1066-48ae-8b05-c70dd9657482.png)
