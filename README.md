# Marketing-Analytics
Marketing Analytics on E-commerce dataset using SQL


`use mavenfuzzyfactory; 
select month(ws.created_at) as 'Month', count(distinct ws.website_session_id) as 'Sessions', count(distinct o.order_id) as 'Orders'
from website_sessions ws 
left join orders o 
on ws.website_session_id=o.website_session_id
where year(ws.created_at)=2012
group by  month(ws.created_at);`

![test](https://user-images.githubusercontent.com/113862057/192161698-a3c17943-0f78-4ea3-abff-8ace7dec9cb5.png)
