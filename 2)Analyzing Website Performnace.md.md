### **Analyzing Website Traffic Sources**

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
All  traffic all comes in through the homepage 
There has to be improvements made to attract traffic sources from alternate entry pages


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

