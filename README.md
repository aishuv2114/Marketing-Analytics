
## Marketing Analytics on E-commerce dataset using SQL

### **Analyzing Website Traffic Sources**

**Objective**  
Traffic source analysis is about understanding where your customers are coming from and which channels are driving the highest quality traffic

1)The leadership team would like to know where the bulk of the website sessions are coming by UTM source , campaign and referring domain 

```sql
use mavenfuzzyfactory; 

select month(ws.created_at) as 'Month', 
count(distinct ws.website_session_id) as 'Sessions', 
count(distinct o.order_id) as 'Orders'
from website_sessions ws 
left join orders o 
on ws.website_session_id=o.website_session_id
where year(ws.created_at)=2012
group by  month(ws.created_at);
```
**Output**  
![Screenshot_10](https://user-images.githubusercontent.com/113862057/192175252-531acdb8-e749-497c-9ed1-8b9cfa6c76db.png)

**Insights & Actions**
- This shows that gsearch nonbrand is the major traffic source. 

2)This query analyzes if gsearch nonbrand sessions are the major drivers of sales

```sql
select count(distinct ws.website_session_id) as 'Sessions',
count(Distinct o.website_session_id) as 'Orders',
count(Distinct o.website_session_id)/count(Distinct ws.website_session_id) as 'CVR'
from website_sessions ws 
left join orders o 
on ws.website_session_id=o.website_session_id
where utm_source='gsearch' 
and utm_campaign='nonbrand'
and ws.created_at<'2012-04-14';

```

![Screenshot_1](https://user-images.githubusercontent.com/113862057/192175590-3b5b22db-8803-46b2-8ec4-8710c2d96915.png)

Analysis- A 

