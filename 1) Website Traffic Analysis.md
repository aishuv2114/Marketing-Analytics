## Marketing Analytics on E-commerce dataset using SQL - Part 1 of 5

### **Analyzing Website Traffic Sources**

**Objective**  
Traffic source analysis is about understanding where your customers are coming from and which channels are driving the highest quality traffic

1)The leadership team would like to know where the bulk of the website sessions are coming by UTM source , campaign and referring domain 

```sql
SELECT utm_source,
       utm_campaign,
       http_referer,
       Count(DISTINCT website_session_id) AS 'Sessions'
FROM   website_sessions
WHERE  created_at <= '2012-04-12'
GROUP  BY utm_source,
          utm_campaign,
          http_referer
ORDER  BY sessions DESC;
```
**Output**  
![Screenshot_10](https://user-images.githubusercontent.com/113862057/192175252-531acdb8-e749-497c-9ed1-8b9cfa6c76db.png)

**Insights & Actions**
- This shows that 97.4% of the traffic source is from gsearch nonbrand.  

2)The leadership team would like to analyze if those sessions are driving sales(Conversion Rate from session to order).

```sql
SELECT Count(DISTINCT ws.website_session_id) AS 'Sessions',
       Count(DISTINCT o.website_session_id)  AS 'Orders',
       Count(DISTINCT o.website_session_id) / Count(DISTINCT ws.website_session_id) AS 'CVR'
FROM   website_sessions ws
       LEFT JOIN orders o
              ON ws.website_session_id = o.website_session_id
WHERE  utm_source = 'gsearch'
       AND utm_campaign = 'nonbrand'
       AND ws.created_at < '2012-04-14'; 

```

**Output**  
![Screenshot_1](https://user-images.githubusercontent.com/113862057/192175590-3b5b22db-8803-46b2-8ec4-8710c2d96915.png)

**Insights & Actions**
- The expected CVR is atleast 4% to make numbers work, based on what the company is paying for clicks. 
- The leadership team has decided to dial down the search bids a bit

3)Post the bid down, the marketing director would like to see the gsearch nonbrand trended session volume, by week, to see if the bid changes have caused volume to drop

```sql
SELECT Date(Min(created_at)),
       Count(DISTINCT website_session_id)
FROM   website_sessions ws
WHERE  utm_source = 'gsearch'
       AND utm_campaign = 'nonbrand'
       AND ws.created_at < '2012-05-10'
GROUP  BY Week(created_at); 
```

**Output**  
![Screenshot_2](https://user-images.githubusercontent.com/113862057/192181958-96a264ee-7879-491f-b3ad-2854c4e1f6f3.png)

**Insights & Actions**
- The gsearch nonbrand is sensitive to bid changes. 
- The executive team wants maximum volume, and want to drill into device level performance for gsearch nonbrand

4)The marketing director would like a conversion rate analysis, from session to order by device type. 

```sql
SELECT device_type,
       Count(DISTINCT ws.website_session_id) AS 'Sessions',
       Count(DISTINCT o.website_session_id)  AS 'Orders',
       Count(DISTINCT o.website_session_id) / Count(DISTINCT ws.website_session_id) AS 'CVR'
FROM   website_sessions ws
       LEFT JOIN orders o
              ON ws.website_session_id = o.website_session_id
WHERE  utm_source = 'gsearch'
       AND utm_campaign = 'nonbrand'
       AND ws.created_at < '2012-05-11'
GROUP  BY device_type; 
```

**Output**  
![Screenshot_3](https://user-images.githubusercontent.com/113862057/192182668-38e19a1d-ad3e-4608-8df6-e291d8600bec.png)

**Insights & Actions**
- The desktop performance for gsearch nonbrand is better than on mobile. 
- The company opts to increase the bids on desktop

5)The marketing team wants to do a post analysis to gauge the results of the bid up 

```sql
SELECT Date(Min(created_at)) AS 'Week_Start_date',
       Count(DISTINCT CASE
                        WHEN device_type = 'desktop' THEN website_session_id
                      end)   AS dtop_sessions,
       Count(DISTINCT CASE
                        WHEN device_type = 'mobile' THEN website_session_id
                      end)   AS mob_sessions
FROM   website_sessions ws
WHERE  utm_source = 'gsearch'
       AND utm_campaign = 'nonbrand'
       AND ws.created_at >= '2012-04-15'
       AND ws.created_at < '2012-06-09'
GROUP  BY Week(created_at); 
```

**Output**  
![Screenshot_4](https://user-images.githubusercontent.com/113862057/192183278-23feac7f-5c8a-4951-ac3f-eb1405f25953.png)


**Insights & Actions**
Desktop performance is looking strong thanks to the bid changes the company made based on the previous conversion analysis.


