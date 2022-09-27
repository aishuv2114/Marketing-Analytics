## Marketing Analytics on E-commerce dataset using SQL - Part 3 of 5

### **Analyzing Channel Portfolios**

**Objective**  
Analyzing a portfolio of marketing channels is about bidding efficiently and using data to maximize the effectiveness of your marketing budget.
This helps in understanding which marketing channels are driving the most sessions and orders through the website,optimizing bids and allocating marketing spend across a multi-channel portfolio to achieve maximum performance.

1)The oragnization has launched another paid search channel-bsearch since Aug 22,2012.This marketing director has a data request to view weekly trended sessions since then and compare it to gsearch nonbrand

```sql
WITH gsearch_nonbrand_sessions
     AS (SELECT Min(Date(created_at))  AS week_start_date,
                Count(DISTINCT website_session_id) AS gsearch_sessions
         FROM   website_sessions
         WHERE  utm_source = 'gsearch'
                AND utm_campaign = 'nonbrand'
                AND ( created_at > '2012-08-22'
                      AND created_at < '2012-11-29' )
         GROUP  BY Week(created_at)),
     bsearch_sessions
     AS (SELECT Min(Date(created_at)) AS week_start_date,
                Count(DISTINCT website_session_id) AS bsearch_sessions
         FROM   website_sessions
         WHERE  utm_source = 'bsearch'
                AND utm_campaign = 'nonbrand'
                AND ( created_at > '2012-08-22'
                      AND created_at < '2012-11-29' )
         GROUP  BY Week(created_at))
SELECT g.week_start_date,
       gsearch_sessions,
       bsearch_sessions
FROM   gsearch_nonbrand_sessions g
       LEFT JOIN bsearch_sessions b
              ON g.week_start_date = b.week_start_date; 
```


 **Output**  
![Screenshot_15](https://user-images.githubusercontent.com/113862057/192191674-88e02588-9feb-4a69-83ce-5904279d75eb.png)

 **Insights & Actions**  
Bsearch tends to get roughly a third the traffic of gsearch

2)The marketing director would like to learn more about the bsearch nonbrand campaign and would like to see the percentage of traffic coming on Mobile , and compare that to gsearch

```sql
SELECT utm_source,
       Count(DISTINCT website_session_id),
       Count(CASE WHEN device_type = 'mobile' THEN website_session_id end) AS mobile_sessions,
       Count(CASE WHEN device_type = 'mobile' THEN website_session_id end) / Count(DISTINCT website_session_id) AS pct_mobile
FROM   website_sessions
WHERE  utm_source IN ( 'bsearch', 'gsearch' )
       AND utm_campaign = 'nonbrand'
       AND ( created_at > '2012-08-22'
             AND created_at < '2012-11-30' )
GROUP  BY utm_source; 
```

 **Output**  
![Screenshot_16](https://user-images.githubusercontent.com/113862057/192192119-d746123c-e6da-4039-9f5d-0242a74fbd7b.png)

3)The executive team wants a report on nonbrand conversion rates from session to order for gsearch and bsearch, aslicing the data by device type from
August 22 to September 18

```sql
SELECT device_type,
       utm_source,
       Count(DISTINCT ws.website_session_id) AS sessions,
       Count(DISTINCT o.order_id) AS orders,
       Count(DISTINCT o.order_id) / Count(DISTINCT ws.website_session_id) AS 'conv_rate'
FROM   website_sessions ws
       LEFT JOIN orders o
              ON ws.website_session_id = o.website_session_id
WHERE  utm_campaign = 'nonbrand'
       AND ( ws.created_at >= '2012-08-22'
             AND ws.created_at <= '2012-09-19' )
GROUP  BY device_type,
          utm_source; 
```   

**Output**  
![Screenshot_17](https://user-images.githubusercontent.com/113862057/192192700-9c0d08bf-a26c-4f04-b49f-03654347df44.png)

 **Insights & Actions**  
The channels don’t perform identically,the bids can be differentiated in order to optimize the overall paid marketing budget
bsearch can be bid down based on its under performance


3)Based on the previous analysis, the company has bid down bsearch nonbrand on Dec 2nd,2012.The team has now requested for weekly session volume for gsearch and bsearch nonbrand, broken down by device, since November 4th.

```sql
SELECT Min(Date(created_at)) AS 'Week_Start_Date',
       Count(DISTINCT CASE WHEN utm_source = 'gsearch' AND device_type = 'desktop' THEN website_session_id END) AS g_dtop_sessions,
       Count(DISTINCT CASE WHEN utm_source = 'bsearch' AND device_type = 'desktop' THEN website_session_id END)  AS b_dtop_sessions,
       Count(DISTINCT CASE WHEN utm_source = 'bsearch' AND device_type = 'desktop' THEN website_session_id END) / Count(DISTINCT CASE WHEN utm_source = 'gsearch'
                                                   AND device_type = 'desktop'  THEN website_session_id  END) AS 'b_pct_g_dtop',
       Count(DISTINCT CASE WHEN utm_source = 'gsearch' AND device_type = 'mobile' THEN website_session_id END) AS m_dtop_sessions,
       Count(DISTINCT CASE WHEN utm_source = 'bsearch' AND device_type = 'mobile' THEN website_session_id END) AS m_dtop_sessions,
       Count(DISTINCT CASE WHEN utm_source = 'bsearch' AND device_type = 'mobile' THEN website_session_id END) / Count(DISTINCT CASE WHEN utm_source = 'gsearch'
                                                   AND device_type = 'mobile' THEN website_session_id END) AS 'b_pct_g_mob'
FROM   website_sessions
WHERE  utm_campaign = 'nonbrand'
       AND created_at > '2012-11-04'
       AND created_at < '2012-12-22'
GROUP  BY Week(created_at); 
```

**Output**  
![Screenshot_18](https://user-images.githubusercontent.com/113862057/192407352-de7ba2d6-bde2-4587-9ab9-24f94e39b00e.png)

 **Insights & Actions**  
 - bsearch traffic dropped off a bit after the bid down . 
 - It also looks like gsearch was down too after Black Friday and Cyber Monday, but bsearch dropped even more
 
 4)A potential investor is asking if the company is building momentum with its brand or if the company will need to keep relying on paid traffic.
The executive team has a request to pull pull organic search, direct type in, and paid brand search sessions by month , and show those sessions
as a % of paid search nonbrand

```sql
SELECT Month(created_at)  AS Mnth,
       Count(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN website_session_id END)  AS 'nonbrand',
       Count(DISTINCT CASE WHEN utm_campaign = 'brand' THEN website_session_id END)  AS 'brand',
       Count(DISTINCT CASE WHEN utm_campaign = 'brand' THEN website_session_id END) / Count(DISTINCT CASE WHEN utm_campaign = 'nonbrand'
                                            THEN  website_session_id END) AS 'brand_pct_of_nonbrand',
       Count(DISTINCT CASE WHEN utm_content IS NULL AND http_referer IS NULL THEN website_session_id END) AS 'Direct_Type_In',
       Count(DISTINCT CASE WHEN utm_content IS NULL AND http_referer IS NULL THEN website_session_id END) / Count(DISTINCT CASE
                                              WHEN utm_campaign = 'nonbrand' THEN website_session_id END) AS 'Direct_pct_of_nonbrand',
       Count(DISTINCT CASE WHEN utm_content IS NULL AND http_referer IS NOT NULL THEN website_session_id END)  AS 'Organic',
       Count(DISTINCT CASE WHEN utm_content IS NULL AND http_referer IS NOT NULL THEN website_session_id END) / Count(DISTINCT CASE WHEN utm_campaign = 'nonbrand'
                                            THEN website_session_id END) AS 'organic_pct_of_nonbrand'
FROM   website_sessions
WHERE  created_at <= '2012-12-23'
GROUP  BY Month(created_at); 
```

**Output**  
![Screenshot_19](https://user-images.githubusercontent.com/113862057/192410114-b9bd7a9b-1f19-4687-bf9c-e9387deac07b.png)


 **Insights & Actions**  
The analysis shows that direct and organic volumes are growing. 


