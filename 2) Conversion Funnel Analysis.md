## Marketing Analytics on E-commerce dataset using SQL - Part 2 of 4

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
