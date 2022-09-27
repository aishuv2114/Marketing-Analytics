## Marketing Analytics on E-commerce dataset using SQL - Part 4 of 5

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
 It looks like the CTR from the /cart page didn’t go down and the products per order, AOV, and revenue per /cart session are all up slightly since the
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
 
 

 

