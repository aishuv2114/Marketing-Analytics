# Overview

This is an online course which analyzes an e-commerce dataset  
I have used a variety of SQL commands and features including, but not limited to - Joins, Sub-query, CTE, Temporary Table, Windows Functions, Aggregate Functions, Date Functions, String Functions and CASE statements  
Under each business question I have provided the SQL query, output and my insights

# Focus Areas

[Website Traffic Analysis](#analyzing-website-traffic-sources)  
[Analyzing Website Performance](#analyzing-website-performance)  
[Conversion Funnel Analysis](#conversion-funnel-analysis)  
[Marketing Channel Analysis](#marketing-channel-analysis)  
[Product Analysis](#product-analysis)  
[User Analysis](#user-analysis)  


# ER Diagram
![Screenshot_1](https://user-images.githubusercontent.com/113862057/193383982-24d2d29b-9788-47a4-b148-1dc8823570a1.png)



# Analyzing Website Traffic Sources

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
- The business can dial down bids for gsearch nonbrand to see check if there is a drop in volume

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
- A device level analysis could be done to check which device has a stronger impact on volume to make bid changes accordingly.

4)This query does a conversion rate analysis, from session to order by device type. 

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
- The business can increase the bids on desktop

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

### **Analyzing Website Performance**

**Objective**  
Website content analysis is about understanding which pages are seen the most by users, to identify where to focus on improving the business

1)The website manager has a data request to pull the most viewed website pages ranked by session volume

```sql
SELECT pageview_url,
       Count(DISTINCT website_session_id) AS 'sessions'
FROM   website_pageviews
WHERE  created_at < '2012-06-09'
GROUP  BY pageview_url
ORDER  BY Count(DISTINCT website_session_id) DESC; 
```

**Output**  
![Screenshot_5](https://user-images.githubusercontent.com/113862057/192184728-af8e9ca4-cbb3-4531-b00e-5f531880e6c2.png)

**Insights & Actions**  
The homepage, the products page,and the Mr. Fuzzy page get the bulk of the traffic.

2)The website manager wants to confirm where the users are hitting the site. She wants to rank all entry pages based on entry volume.

```sql
WITH top_entry_ids
     AS (SELECT website_session_id,
                Min(website_pageview_id) AS 'min_pageview_id'
         FROM   website_pageviews wp
         WHERE  created_at < '2012-06-12'
         GROUP  BY website_session_id),
     pageview_url_name
     AS (SELECT t.website_session_id,
                pageview_url
         FROM   top_entry_ids t
                JOIN website_pageviews wp
                  ON t.min_pageview_id = wp.website_pageview_id)
SELECT pageview_url,
       Count(DISTINCT website_session_id)
FROM   pageview_url_name; 
```

**Output**   
![Screenshot_6](https://user-images.githubusercontent.com/113862057/192185837-e68e34ac-24f5-499c-9f3a-888f1fd3d9a0.png)

**Insights & Actions**  
All  traffic comes in through the homepage 
Improvements can be made to attract traffic sources from alternate entry pages to increase volume


3)The website manager wants to see bounce rates for traffic landing on the homepage- Sessions ,Bounced Sessions , and Bounce Rate.

```sql
WITH total_sessions
     AS (SELECT website_session_id
         FROM   website_pageviews
         WHERE  created_at < '2012-06-14'),
     bounced_sessions
     AS (SELECT website_session_id,
                Count(DISTINCT website_pageview_id)
         FROM   website_pageviews
         WHERE  created_at < '2012-06-14'
         GROUP  BY website_session_id
         HAVING Count(DISTINCT website_pageview_id) = 1)
SELECT Count(DISTINCT ts.website_session_id) AS 'Sessions',
       Count(DISTINCT bs.website_session_id) AS 'Bounced Sessions',
       Count(DISTINCT bs.website_session_id) / Count(DISTINCT ts.website_session_id) AS 'Bounce_Rate'
FROM   total_sessions ts
       LEFT JOIN bounced_sessions bs
              ON ts.website_session_id = bs.website_session_id; 
```

**Output**  
![Screenshot_7](https://user-images.githubusercontent.com/113862057/192186393-638263a1-3482-410c-8923-b97e63271c8a.png)


**Insights & Actions**  
There is a 60% bounce rate which is very high for a paid search 
The website team could work on a custom landing page for search, and set up an experiment to see if the new page does better. 

4)Based on the bounce rate analysis, the website team has created a new custom landing page ( (/lander 1 ) in a 50/50 test against the homepage ((/home for our gsearch nonbrand traffic.The team would like to view bounce rates for the two groups to evaluate the new page baseb on a common time frame.

```sql
WITH total_sessions
     AS (SELECT pageview_url,
                Count(DISTINCT ws.website_session_id) AS total_sessions
         FROM   website_sessions ws
                LEFT JOIN website_pageviews wp
                       ON ws.website_session_id = wp.website_session_id
         WHERE  ( ws.created_at >= '2012-06-19'
                  AND ws.created_at < '2012-07-28' )
                AND utm_source = 'gsearch'
                AND utm_campaign = 'nonbrand'
                AND ( pageview_url = '/home'
                       OR pageview_url = '/lander-1' )
         GROUP  BY pageview_url),
     bounced_sessions_tbl
     AS (SELECT wp.website_session_id,
                Count(DISTINCT website_pageview_id) AS 'pageview_id'
         FROM   website_pageviews wp
                JOIN website_sessions ws
                  ON ws.website_session_id = wp.website_session_id
         WHERE  ( ws.created_at >= '2012-06-19'
                  AND ws.created_at < '2012-07-28' )
                AND utm_source = 'gsearch'
                AND utm_campaign = 'nonbrand'
         GROUP  BY ws.website_session_id
         HAVING Count(DISTINCT website_pageview_id) = 1),
     bounced_sessions
     AS (SELECT pageview_url                         AS 'landing_page',
                Count(DISTINCT s.website_session_id) AS 'bounced_sessions'
         FROM   bounced_sessions_tbl s
                JOIN website_pageviews p
                  ON p.website_session_id = s.website_session_id
         GROUP  BY pageview_url)
SELECT bs.landing_page,
       ts.total_sessions,
       bs.bounced_sessions,
       bs.bounced_sessions / ts.total_sessions AS 'Bounce_rate'
FROM   bounced_sessions bs
       JOIN total_sessions ts
         ON bs.landing_page = ts.pageview_url; 
```

**Output**  
![Screenshot_8](https://user-images.githubusercontent.com/113862057/192187062-acb7c7b4-fd27-4543-8ccf-e38ee9423801.png)


**Insights & Actions**  
- The custom lander has a lower bounce rate success
- As a next step, the business can get the campaigns updated so that all nonbrand paid traffic is pointing to the new page. 


5)The website manager has a request to pull the volume of paid search nonbrand traffic landing on /home and /lander 1, trended weekly since June 1st and also pull the  overall paid search bounce rate trended weekly ? 

```sql
WITH sessions_count
     AS (SELECT Min(Date(wp.created_at)) AS week_start_date,
                Count(CASE
                        WHEN pageview_url = '/home' THEN wp.website_session_id
                      END)               AS 'home_sessions',
                Count(CASE
                        WHEN pageview_url = '/lander-1' THEN
                        wp.website_session_id
                      END)               AS 'lander_sessions'
         FROM   website_sessions ws
                LEFT JOIN website_pageviews wp
                       ON ws.website_session_id = wp.website_session_id
         WHERE  utm_source = 'gsearch'
                AND utm_campaign = 'nonbrand'
                AND ( wp.created_at >= '2012-06-01'
                      AND wp.created_at < '2012-08-31' )
         GROUP  BY Week(wp.created_at)),
     total_sessions
     AS (SELECT Min(Date(wp.created_at))              AS week_start_date,
                Count(DISTINCT wp.website_session_id) AS 'total_sessions'
         FROM   website_sessions ws
                LEFT JOIN website_pageviews wp
                       ON ws.website_session_id = wp.website_session_id
         WHERE  utm_source = 'gsearch'
                AND utm_campaign = 'nonbrand'
                AND ( wp.created_at >= '2012-06-01'
                      AND wp.created_at < '2012-08-31' )
         GROUP  BY Week(wp.created_at)),
     bounced_sessions_tbl
     AS (SELECT Min(Date(wp.created_at)) AS week_start_date,
                wp.website_session_id
         FROM   website_sessions ws
                LEFT JOIN website_pageviews wp
                       ON ws.website_session_id = wp.website_session_id
         WHERE  utm_source = 'gsearch'
                AND utm_campaign = 'nonbrand'
                AND ( wp.created_at >= '2012-06-01'
                      AND wp.created_at < '2012-08-31' )
         GROUP  BY Date(wp.created_at),
                   wp.website_session_id
         HAVING Count(DISTINCT website_pageview_id) = 1),
     bounced_sessions
     AS (SELECT Min(week_start_date)               AS week_start_date,
                Count(DISTINCT website_session_id) AS 'Bounced_sessions'
         FROM   bounced_sessions_tbl
         GROUP  BY Week(week_start_date)),
     bounce_rate
     AS (SELECT bs.week_start_date,
                bounced_sessions / total_sessions AS bounce_rate
         FROM   bounced_sessions bs
                JOIN total_sessions ts
                  ON ts.week_start_date = bs.week_start_date)
SELECT br.week_start_date,
       bounce_rate,
       home_sessions,
       lander_sessions
FROM   bounce_rate br
       JOIN sessions_count sc
         ON br. week_start_date = sc.week_start_date; 
```


**Output**  
![Screenshot_9](https://user-images.githubusercontent.com/113862057/192188098-90fe0901-27cf-4c5b-b264-41126cb52ce1.png)


**Insights & Actions**  
The overall bounce rate has come down over time



### **Conversion Funnel Analysis**

**Objective**  
Conversion funnel analysis is about understanding and optimizing each step of your user’s experience on their journey toward purchasing your products

1)The website manager would like to understand where the gsearch visitors are lost between the new /lander 1 page and placing an order and would like to see a full conversion funnel, analyzing how many customers make it to each step,starting with /lander 1 and  all the way to the thank you page,since August 5th

```sql
SELECT Count(DISTINCT website_session_id) AS 'sessions',
       Count(CASE WHEN to_products=1 THEN website_session_id ELSE NULL end ) to_products,
       Count(CASE WHEN to_mrfuzzy=1 THEN website_session_id ELSE NULL end ) to_mrfuzzy,
       Count(CASE WHEN to_cart=1 THEN website_session_id ELSE NULL end ) to_cart,
       Count(CASE WHEN to_shipping=1 THEN website_session_id ELSE NULL end ) to_shipping,
       Count(CASE WHEN to_billing=1 THEN website_session_id ELSE NULL end ) to_billing,
       Count(CASE WHEN to_thankyou=1 THEN website_session_id ELSE NULL end ) to_thankyou
FROM   (
                SELECT   website_session_id,
                         pageview_url,
                         CASE WHEN pageview_url='/products' THEN 1 ELSE 0 end AS to_products,
                         CASE WHEN pageview_url='/the-original-mr-fuzzy' THEN 1 ELSE 0 end AS to_mrfuzzy,
                         CASE WHEN pageview_url='/cart' THEN 1 ELSE 0 end AS to_cart,
                         CASE WHEN pageview_url='/shipping' THEN 1 ELSE 0 end AS to_shipping,
                         CASE WHEN pageview_url='/billing' THEN 1  ELSE 0 end AS to_billing,
                         CASE WHEN pageview_url='/thank-you-for-your-order' THEN 1 ELSE 0 end AS to_thankyou
                FROM     (
                                SELECT website_session_id,
                                       pageview_url
                                FROM   website_pageviews
                                WHERE  website_session_id IN
                                                              (
                                                              SELECT DISTINCT wp.website_session_id
                                                              FROM            website_sessions ws
                                                              LEFT JOIN       website_pageviews wp
                                                              ON              ws.website_session_id=wp.website_session_id
                                                              WHERE           utm_source='gsearch'
                                                              AND             utm_campaign='nonbrand'
                                                              AND             (
                                                                                              wp.created_at>='2012-08-05'
                                                                              AND             wp.created_at<'2012-09-05')
                                                              AND             pageview_url='/lander-1' ))a
                GROUP BY website_session_id,
                         pageview_url)b;
```

**Click Rates**  

```sql
WITH final_output
     AS (SELECT Count(DISTINCT website_session_id) AS 'sessions',
                Count(CASE WHEN to_products = 1 THEN website_session_id ELSE NULL END) to_products,
                Count(CASE WHEN to_mrfuzzy = 1 THEN website_session_id  ELSE NULL END) to_mrfuzzy,
                Count(CASE WHEN to_cart = 1 THEN website_session_id ELSE NULL END) to_cart,
                Count(CASE WHEN to_shipping = 1 THEN website_session_id ELSE NULL END) to_shipping,
                Count(CASE WHEN to_billing = 1 THEN website_session_id ELSE NULL END) to_billing,
                Count(CASE WHEN to_thankyou = 1 THEN website_session_id ELSE NULL END) to_thankyou
         FROM   (SELECT website_session_id,
                        pageview_url,
                        CASE WHEN pageview_url = '/products' THEN 1 ELSE 0 END AS to_products,
                        CASE WHEN pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS to_mrfuzzy,
                        CASE WHEN pageview_url = '/cart' THEN 1 ELSE 0 END AS to_cart,
                        CASE WHEN pageview_url = '/shipping' THEN 1 ELSE 0 END AS to_shipping,
                        CASE WHEN pageview_url = '/billing' THEN 1  ELSE 0 END AS to_billing,
                        CASE WHEN pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS to_thankyou
                 FROM   (SELECT website_session_id,
                                pageview_url
                         FROM   website_pageviews
                         WHERE  website_session_id IN
                                (SELECT DISTINCT wp.website_session_id
                                 FROM   website_sessions ws
                                        LEFT JOIN website_pageviews wp
                                               ON
                                ws.website_session_id = wp.website_session_id
                                                       WHERE
                                utm_source = 'gsearch'
                                AND
                                utm_campaign = 'nonbrand'
                                                              AND (
                                wp.created_at >= '2012-08-05'
                                AND
                                wp.created_at < '2012-09-05' )
                                    AND
                                pageview_url = '/lander-1'))a
                 GROUP  BY website_session_id,
                           pageview_url)b)
SELECT to_products / sessions   AS lander_click_rt,
       to_mrfuzzy / to_products AS products_click_rt,
       to_cart / to_mrfuzzy     AS mrfuzzy_click_rt,
       to_shipping / to_cart    AS cart_click_rt,
       to_billing / to_shipping AS shipping_click_rt,
       to_thankyou / to_billing AS billing_click_rt
FROM   final_output; 
```

 **Output**                         
![Screenshot_11](https://user-images.githubusercontent.com/113862057/192189183-760753f6-d918-4e1f-8886-75c9cdad9ef3.png)
![Screenshot_13](https://user-images.githubusercontent.com/113862057/192190442-e7add925-c100-4861-b104-4e596fe80dc2.png)

**Insights & Actions**  
The lander, Mr. Fuzzy page ,and the billing page have the lowest click rates.


2)The team has tested an updated billing page based on the funnel analysis. There is a new request to see whether /billing 2 is doing any better than the original /billing page and what % of sessions on those pages end up placing an order . This test is for all traffic, not just for the search visitors

```sql
SELECT Date(Min(created_at)),
       Min(website_pageview_id)
FROM   website_pageviews
WHERE  pageview_url = '/billing-2';

SELECT pageview_url,
       Count(DISTINCT wp.website_session_id) AS 'sessions',
       Count(DISTINCT order_id) AS 'orders',
       Count(DISTINCT order_id) / Count(DISTINCT wp.website_session_id) AS 'billing_to_order_rt'
FROM   website_pageviews wp
       LEFT JOIN orders o
              ON wp. website_session_id = o.website_session_id
WHERE  ( wp.created_at >= '2012-09-10'
         AND wp.created_at < '2012-11-10' )
       AND pageview_url LIKE '%billing%'
GROUP  BY pageview_url; 
```

 **Output**  
 ![Screenshot_14](https://user-images.githubusercontent.com/113862057/192190985-65061cbb-5349-4d0f-b118-ec7123053e27.png)
 
 **Insights & Actions**  
The new version of the billing page is doing a much better job converting customers
As a next step,the Engineering team could roll this out to all customers



### **Marketing Channel Analysis**

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




### **Product Analysis**

**Objective**
Analyzing product sales helps you understand how each product contributes to your business, and how product launches impact the overall portfolio

1)The organization is about to launch a new product, and they would like to do a deep dive on the current flagship product.They want to pull yearly trends to date for number of sales , total revenue , and total margin generated for the business

```sql
SELECT Year(created_at),      
       Count(DISTINCT order_id)  AS 'Number of Sales',
       Sum(price_usd)            AS 'total_revenue',
       Sum(price_usd - cogs_usd) AS 'total_margin'
FROM   orders
GROUP  BY Year(created_at);
        
```

**Output**  
![Screenshot_21](https://user-images.githubusercontent.com/113862057/192411215-65ab581a-1066-48ae-8b05-c70dd9657482.png)


2)The organization has launched a second product  on January 6 th . They would like to see monthly order volume , overall conversion rates , revenue per session , and a breakdown of sales by product , all for the time period since April 1, 2013

```sql
SELECT Year(ws.created_at)  AS  'yr',
       Month(ws.created_at) AS  'Mnth',
       Count(DISTINCT order_id) AS 'Orders',
       Count(DISTINCT o.order_id) / Count(DISTINCT ws.website_session_id) AS  'CVR',
       Sum(price_usd) / Count(DISTINCT ws.website_session_id)  AS 'Revenue_Per_Session',
       Count(DISTINCT CASE WHEN primary_product_id = 1 THEN o.order_id END) AS product_one_orders,
       Count(DISTINCT CASE WHEN primary_product_id = 2 THEN o.order_id END)  AS product_two_orders
FROM   website_sessions ws
       LEFT JOIN orders o
              ON ws.website_session_id = o.website_session_id
WHERE  Date(ws.created_at) >= '2012-04-01'
GROUP  BY Year(ws.created_at),
          Month(ws.created_at); 
```        

**Output**  
![Screenshot_22](https://user-images.githubusercontent.com/113862057/192411564-21b2080e-5292-4c46-9f43-01b656aecc1a.png)


 **Insights & Actions**  
 - This confirms that our conversion rate and revenue per session are improving over time
 - However,further analysis needs to be done to see if the increase is due to the new product launch or just a continuation of overall business improvements
 
 3)The website manager has request to see clickthrough rates from /products since the new product launch on January 6 th 2013 , by product, and compare to the 3 months leading up to launch as a baseline.
 
 ```sql
 WITH sessions
     AS (SELECT Date(ws.created_at)          AS created_at,
                Count(ws.website_session_id) AS 'Sessions'
         FROM   website_sessions ws
                LEFT JOIN website_pageviews wp
                       ON ws.website_session_id = wp.website_session_id
         WHERE  Date(ws.created_at) > '2012-10-06'
                AND Date(ws.created_at) <= '2013-04-06'
                AND pageview_url LIKE '%products%'
         GROUP  BY Date(ws.created_at)),
     sessions_to_products_page
     AS (SELECT Date(ws.created_at)          AS created_at,
                Count(ws.website_session_id) AS 'to_next_page'
         FROM   website_sessions ws
                LEFT JOIN website_pageviews wp
                       ON ws.website_session_id = wp.website_session_id
         WHERE  Date(ws.created_at) > '2012-10-06'
                AND Date(ws.created_at) <= '2013-04-06'
                AND ( pageview_url LIKE '%fuzzy%'
                       OR pageview_url LIKE '%love-bear%' )
         GROUP  BY Date(ws.created_at)),
     sessions_to_fuzzy
     AS (SELECT Date(ws.created_at)          AS created_at,
                Count(ws.website_session_id) AS 'to_mrfuzzy'
         FROM   website_sessions ws
                LEFT JOIN website_pageviews wp
                       ON ws.website_session_id = wp.website_session_id
         WHERE  Date(ws.created_at) > '2012-10-06'
                AND Date(ws.created_at) <= '2013-04-06'
                AND ( pageview_url LIKE '%fuzzy%' )
         GROUP  BY Date(ws.created_at)),
     sessions_to_lovebear
     AS (SELECT Date(ws.created_at)          AS created_at,
                Count(ws.website_session_id) AS 'to_lovebear'
         FROM   website_sessions ws
                LEFT JOIN website_pageviews wp
                       ON ws.website_session_id = wp.website_session_id
         WHERE  Date(ws.created_at) > '2012-10-06'
                AND Date(ws.created_at) <= '2013-04-06'
                AND ( pageview_url LIKE '%love-bear%' )
         GROUP  BY Date(ws.created_at))
SELECT CASE
         WHEN s.created_at > '2012-10-06'
              AND s.created_at < '2013-01-06' THEN "pre"
         ELSE "post"
       END               AS created_at,
       Sum(sessions)     AS 'Sessions',
       Sum(to_next_page) AS to_next_page,
       Sum(to_mrfuzzy)   AS to_mrfuzzy,
       Sum(to_lovebear)  AS to_lovebear
FROM   sessions s
       LEFT JOIN sessions_to_products_page sp
              ON s.created_at = sp.created_at
       LEFT JOIN sessions_to_fuzzy sf
              ON sp.created_at = sf.created_at
       LEFT JOIN sessions_to_lovebear sl
              ON sf.created_at = sl.created_at
GROUP  BY created_at; 
 ```
 
 **Output**  
 ![Screenshot_23](https://user-images.githubusercontent.com/113862057/192412927-2a704d9b-3d1d-4a47-9ca9-6821d7800315.png) 
 
 
 **Insights**  
- The percent of /products pageviews that clicked to Mr. Fuzzy has gone down since the launch of the Love Bear,but the overall clickthrough rate has gone up, so it seems to be generating additional product interest overall.

4)On September 25th, the organization has started giving customers the option to add a 2 nd product while on the /cart page . The executive team has a request 
to compare the month before vs the month after the change ,they would like to see CTR from the /cart page ,Avg Products per Order , AOV , and overall revenue per
/cart page view
 
 
 ```sql
 WITH order_details
     AS (SELECT CASE WHEN wp.created_at < '2013-09-25' THEN 'Pre'  ELSE 'Post' END  AS time_period,
                Count(DISTINCT CASE WHEN pageview_url LIKE '%cart%' THEN  wp.website_session_id  END)  AS Cart_Sessions',
                Count(DISTINCT CASE WHEN pageview_url LIKE '%shipping%' THEN wp.website_session_id END)  AS 'Clickthroughs',
                Count(DISTINCT CASE WHEN pageview_url LIKE '%shipping%' THEN wp.website_session_id END) / Count(DISTINCT CASE
                                                       WHEN pageview_url LIKE '%cart%' THEN wp.website_session_id END) AS  'cart_ctr',
                Count(DISTINCT o.order_id) AS  'Orders_placed',
                Count(DISTINCT oi.order_item_id)  AS  'Products_Purchased',
                Count(DISTINCT oi.order_item_id) / Count(DISTINCT o.order_id) AS 'products_per_order'
         FROM   website_pageviews wp
                LEFT JOIN orders o
                       ON o.website_session_id = wp.website_session_id
                LEFT JOIN order_items oi
                       ON o.order_id = oi.order_id
         WHERE  ( Date(wp.created_at) BETWEEN '2013-08-25' AND '2013-10-24' )
         GROUP  BY 1),
     revenue_details
     AS (SELECT CASE WHEN o.created_at < '2013-09-25' THEN 'Pre' ELSE 'Post' END  AS time_period,
                Sum(price_usd)                     AS 'Revenue',
                Sum(price_usd) / Count(o.order_id) AS 'AOV'
         FROM   orders o
         WHERE  ( Date(o.created_at) BETWEEN '2013-08-24' AND '2013-10-24' )
         GROUP  BY 1)
SELECT o.time_period,
       Sum(cart_sessions)                AS 'Cart_Sessions',
       Sum(clickthroughs)                AS 'Clickthroughs',
       Sum(cart_ctr)                     AS 'Cart_Ctr',
       Sum(orders_placed)                AS 'Orders_Placed',
       Sum(products_purchased)           AS 'Products_Purchased',
       Sum(products_per_order)           AS 'Products_Per_Order',
       Sum(revenue)                      AS 'Revenue',
       Sum(aov)                          AS 'AOV',
       Sum(revenue) / Sum(cart_sessions) AS Rev_per_cart_sessions
FROM   order_details o
       JOIN revenue_details r
         ON o.time_period = r.time_period
GROUP  BY time_period; 
 ```
 
**Output**  
![Screenshot_24](https://user-images.githubusercontent.com/113862057/192414946-5b6fff1d-ec1b-411f-a546-064ef7cda7bb.png)

 **Insights & Actions**  
The CTR from the /cart page didn’t go down and the products per order, AOV, and revenue per /cart session are all up slightly since the
cross sell feature was added

5)On 12th December, 2013, a third product has been launched -"Birthday Bear". There is a request to run a pre-post analysis comparing the month before and the month after , in terms of session to order conversion rate , AOV, Products per order and Revenue per session

 ```sql
 WITH conv_rate
     AS (SELECT CASE
                  WHEN ws.created_at < '2013-12-12' THEN 'Pre'
                  ELSE 'Post'
                END
                AS
                   time_period,
                Count(DISTINCT o. order_id) /
                Count(DISTINCT ws.website_session_id) AS
                'Session_to_Orders',
                Count(DISTINCT oi.order_item_id) / Count(DISTINCT o.order_id)
                AS
                'Products_per_order',
                Count(ws.website_session_id)
                AS
                   'Sessions',
                Count(o.order_id)
                AS
                   '#Orders'
         FROM   website_sessions ws
                LEFT JOIN orders o
                       ON ws. website_session_id = o.website_session_id
                LEFT JOIN order_items oi
                       ON o.order_id = oi.order_id
         WHERE  Date(ws.created_at) BETWEEN '2013-11-12' AND '2014-01-12'
         GROUP  BY 1),
     revenue
     AS (SELECT CASE
                  WHEN created_at < '2013-12-12' THEN 'Pre'
                  ELSE 'Post'
                END                              AS time_period,
                Sum(price_usd) / Count(order_id) AS 'AOV',
                Sum(price_usd)                   AS revenue
         FROM   orders
         WHERE  Date(created_at) BETWEEN '2013-11-12' AND '2014-01-12'
         GROUP  BY 1)
SELECT c.time_period,
       c.session_to_orders,
       c.products_per_order,
       r.revenue / c.sessions AS 'Rev_per_Session',
       aov
FROM   conv_rate c
       JOIN revenue r
         ON c.time_period = r.time_period; 

 ```
 
 **Output**  
 ![Screenshot_25](https://user-images.githubusercontent.com/113862057/192415416-cf9b216e-30cf-49c1-be4d-07bc2c12a451.png)
 
 


### **User Analysis**

**Objective**
Analyzing repeat visits helps us understand user behavior and identify some of our most valuable customers

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
- A fair number of customers do come back to the site after the first session.


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


4)This query does a comparison of the conversion rates and revenue per session for repeat sessions vs new sessions for 2014 to date.

```sql
SELECT is_repeat_session,
       Count(DISTINCT ws.website_session_id)                              AS
       '#Sessions',
       Count(DISTINCT o.order_id)                                         AS
       '#Orders',
       Count(DISTINCT o.order_id) / Count(DISTINCT ws.website_session_id) AS
       'Conv_Rate',
       Sum(price_usd) / Count(DISTINCT ws.website_session_id)             AS
       'Rev_Per_Session'
FROM   website_sessions ws
       LEFT JOIN orders o
              ON ws.website_session_id = o.website_session_id
WHERE  Date(ws.created_at) >= '2014-01-01'
       AND Date(ws.created_at) <= '2014-11-07'
GROUP  BY is_repeat_session; 
```

**Output**  
![Screenshot_29](https://user-images.githubusercontent.com/113862057/192421261-9b65cfd5-e961-41a0-96aa-ed9929c55196.png)

**Insights**  
- Repeat sessions are more likely to convert , and produce more revenue per session.




 
