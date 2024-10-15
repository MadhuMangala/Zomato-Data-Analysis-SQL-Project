## **Solving Business Problems**
🟢.Q1.Write a query to find the top 5 most frequently ordered dishes by customer called "Arjun Mehta" in the last 1 year.
Note Last one year means (let say today date 2024-09-28 then 
	last 1 year = same period last year on wards)

 ```sql
with cte as (
	SELECT c.customer_id,c.customer_name,o.order_item,
	count(*) as most_orders_cnt,
	dense_rank() over(order by count(*) desc) as rnk 
	FROM customers c
	left join orders o
	using(customer_id)
	where c.customer_name like "%Arjun Mehta%" and o.order_date >= date_add(current_date(),interval -1 year)
	group by 1,2,3
            )
select * from cte where rnk <= 5
```
---
![image](https://github.com/user-attachments/assets/491f0d15-b4f1-4c2a-b85f-fa4d6208e552)
---
🟢.Q2.Identify the time slots during which the most orders are placed, based on 2-hour intervals.

```sql 

select 
concat_ws("-",from_hour,To_hour) as time_slot ,
count(order_id) as order_cnt
from (
 SELECT  order_id,
 case when order_date then floor(hour(order_time)/2)*2 end as from_hour,
 case when order_date then floor(hour(order_time)/2)*2 + 2 end as To_hour
 FROM zomato_project.orders
) x
group by concat_ws("-",from_hour,To_hour)
order by order_cnt  desc
```
---
![image](https://github.com/user-attachments/assets/5f9bf6dc-bc56-45c0-8bca-4a0e52e14e63)

---
🟢.Q3.Find the average order value (AOV) per customer who has placed more than 750 orders. Return: customer_name, aov (average order value).in DESC of AOV.
```sql 
select customer_name,round(avg(total_amount),2) as avg_order_value from customers as c
join orders as o
using(customer_id)
group by customer_name
having count(order_id) >= 750
order by 2 desc
```
---
![image](https://github.com/user-attachments/assets/31d5055d-fdff-4c45-a139-4a1b51a8c120)
---
🟢.Q4.List the customers who have spent more than 100K in total on food orders. Return: customer_name, customer_id.
```sql
select customer_name,customer_id,sum(total_amount) as total_spent 
from customers as c
join orders as o
using(customer_id)
group by customer_name,customer_id
having sum(total_amount) >= 100000
order by 3 desc
```
---
![image](https://github.com/user-attachments/assets/68817b38-5dcd-4650-9803-8d56a3d68f17)

---
🟢.Q5.Write a query to find orders that were placed but not delivered. Return: restaurant_name, city, and the number of not delivered orders.
```sql

SELECT 
r.restaurant_name,r.city,count(order_id) as not_delivered_cnt 
FROM orders as o
left join deliveries as d
using(order_id)
left join restaurants as r
using(restaurant_id)
where order_id is not null and delivery_id is null
group by r.restaurant_name,r.city
order by not_delivered_cnt desc
```
---
![image](https://github.com/user-attachments/assets/57ee3774-54ee-4a65-9861-b0fc2af11509)

---
🟢.Q6.Rank restaurants by their total revenue from the last year. 
Return: restaurant_name, city,total_revenue, and their rank within their city.
```sql

SELECT r.restaurant_name,r.city,sum(total_amount) as total_Revenue,
dense_rank() over(partition by r.city order by sum(total_amount) desc) as Res_rnk 
FROM zomato_project.orders as o
left join restaurants as r
using(restaurant_id)
where year(order_Date) = year(curdate()) - 1
group by r.restaurant_name,r.city,year(order_Date)
order by city,Res_rnk
```
---
![image](https://github.com/user-attachments/assets/0e29bd23-156d-440f-8d1b-b916baa28383)

---
🟢.Q7.Identify the most popular dish in each city based on the number of orders.
Return : city,most popular dish,total order, rnk
```sql
with cte as 
(
	SELECT r.city,o.order_item as most_popular_dish,
	count(order_id) as total_orders, 
	dense_Rank()over(partition by r.city order by count(order_id) desc) rnk
	FROM zomato_project.orders as o
	left join restaurants as r
	using(restaurant_id)
	group by r.city,o.order_item
	order by r.city,rnk
)

select * from cte where rnk = 1
```
---
![image](https://github.com/user-attachments/assets/78fd155d-02de-4a48-8efe-dfbce545e9bd)

---

🟢.Q8.Find customers who haven’t placed an order in 2024 but did in 2023.
```sql
with cte as (SELECT c.customer_id,c.customer_name,o.order_id,year(o.order_date) as yr FROM customers as c
join orders as o
using(customer_id)
)
select distinct customer_name as "2023",customer_id from cte
where yr = 2023
and customer_id not in (select customer_id from cte where yr = 2024)
order by customer_id
```
---
![image](https://github.com/user-attachments/assets/862a36c6-4459-4baa-b178-ba04d4909e17)

---
🟢.Q9.Calculate and compare (restaurant are in both year) the order cancellation(not delivered)rate for each restaurant between the current year
and the previous year.
```sql

with cte as 
(
select o.restaurant_id,count(order_id) as total_orders_2023,
count(case when d.delivery_id is null then 1 else null end) as "not_Delivered_2023" 
-- o.restaurent_id
from orders as o
left join deliveries as d
using(order_id)
where year(order_date) = 2023
group by o.restaurant_id
),
# restaurents order cnt and cancelled cnt in 2024
cte1 as (
select o.restaurant_id,count(order_id) as total_orders_2024,
count(case when d.delivery_id is null then 1 else null end) as "not_Delivered_2024" 
-- o.restaurent_id
from orders as o
left join deliveries as d
using(order_id)
where year(order_date) = 2024
group by o.restaurant_id
)
# compamparision from 2023 vs 2024
select cte.restaurant_id,
round((not_Delivered_2023/total_orders_2023)*100,2) as "2023_cancellation_Date",
round((not_Delivered_2024/total_orders_2024)*100,2) as "2024_cancellation_Date" 
from cte join cte1
using(restaurant_id)
order by cte.restaurant_id
```
---
![image](https://github.com/user-attachments/assets/61ed7251-3604-415b-94ed-982eb45eff3e)

---
🟢.Q11.Calculate each restaurant's growth ratio based on the total number of delivered orders since its
joining.
```sql
with cte as 
(
SELECT r.restaurant_name,o.restaurant_id,
month(o.order_Date) as mnt,year(o.order_date) as yr,count(order_id) as order_cnt
FROM zomato_project.orders as o
join restaurants as r
using(restaurant_id)
join deliveries
using(order_id)
where delivery_status  = "Delivered"
group by 1,2,3,4
order by restaurant_name,yr,mnt
),
cte1 as (
select *,
lag(order_cnt) over(partition by restaurant_name order by yr)as prv_month_Value
from cte
)
select *,
round((((order_cnt-prv_month_Value)/prv_month_Value)*100),2) as Growth_Rate
from cte1
order by restaurant_id 
```
---
![image](https://github.com/user-attachments/assets/eceb82eb-54af-4e36-8770-cd9376f75cd1)

---
🟢.Q12.Segment customers into 'Gold' or 'Silver' groups based on their total spending compared to the
average order value (AOV). If a customer's total spending exceeds the AOV, label them as
'Gold'; otherwise, label them as 'Silver'.
Return: The total number of orders and total revenue for each segment.
```sql
with cte as (
SELECT c.customer_id,c.customer_name,count(order_id)as total_orders,
sum(o.total_amount) as total_spent 
FROM zomato_project.customers as c
left join orders as o
using(customer_id)
group by 1,2
)
,cte1 as ( 
select *,
case
	when total_spent > (select avg(total_orders) as aog from cte) then "Gold"
    else "Silver" end as segment
from  cte
)

select segment,sum(total_spent) as total_revenue,sum(total_orders) as number_of_orders
from cte1
group by segment
```
---
![image](https://github.com/user-attachments/assets/33d8a9f6-6688-4fc8-a546-5564dd86167a)

---
🟢.Q13.Calculate each rider's total monthly earnings, assuming they earn 8% of the order amount.
```sql

SELECT r.rider_id,r.rider_name, year(o.order_date) as yr,
month(o.order_date) as mnt,
sum(o.total_amount) as total_revenue,
round(sum(total_amount*0.08),2)as monthly_income
FROM zomato_project.riders as r
left join deliveries as d
using(rider_id)
left join orders as o
using(order_id)

group by 1,2,3,4
order by 1,2,3,4
```
---
![image](https://github.com/user-attachments/assets/c856031e-14f8-4701-a886-d408c3025580)

---
🟢.Q14.Find the number of 5-star, 4-star, and 3-star ratings each rider has.
Riders receive ratings based on delivery time:
● 5-star: Delivered in less than 15 minutes
● 4-star: Delivered between 15 and 20 minutes
● 3-star: Delivered after 20 minutes
```sql
with cte as (SELECT r.rider_id,d.order_id,o.order_time ,d.delivery_time  FROM riders r
left join  deliveries as d
using(rider_id)
join orders as o
using(order_id)
where d.delivery_status = "Delivered"
)
,
cte1 as (
select rider_id,order_id,
case 
	when order_time then ((extract(hour from order_time)*3600) + (extract(minute from order_time)*60) + (extract(second from order_time)))
    end order_time_in_minutes
,case 
	when delivery_time < order_time then ((extract(hour from delivery_time)+24)*3600 + extract(minute from delivery_time)*60 + extract(second from delivery_time))
    else (extract(hour from delivery_time)*3600 + extract(minute from delivery_time)*60 + extract(second from delivery_time))
   end delivery_time_in_minutes
from cte
),
cte2 as (select *, round((delivery_time_in_minutes - order_time_in_minutes)/60,2) as duration_delivery_time from cte1
)

select rider_id,
count(case when ratings = "5-star" then 1 else null end) as "5-star_rating",
count(case when ratings = "4-star" then 1 else null end) as "4-star_rating",
count(case when ratings = "3-star" then 1 else null end) as "3-star_rating"
from 
(
select *,
	case 
		when duration_delivery_time<15 then "5-star"
        when duration_delivery_time between 15 and 20 then "4-star"
        else "3-star"
		END "ratings"
from cte2
) x
group by rider_id
order by 1
```
---
![image](https://github.com/user-attachments/assets/b5078271-6d5b-42ce-aee3-f55ea4522dfb)

---
🟢.Q15.Analyze order frequency per day of the week and identify the peak day for each restaurant.
```sql

with cte as (
SELECT restaurant_name, 
count(order_id) as order_frequency,dayname(order_date) as "week_Day"
-- dense_Rank() over(partition by restaurant_name,dayname(order_date) order by count(order_id) desc) as flag
FROM zomato_project.orders
join restaurants as r
using(restaurant_id)
group by 1,3
),
cte1 as (
select * 
,
dense_rank() over(partition by restaurant_name order by order_frequency desc)as flag
from cte
)

select * from cte1 where flag = 1
```
---
![image](https://github.com/user-attachments/assets/cab2d2cd-e69e-48dc-b632-70f057ad237f)

---
🟢.Q16Calculate the total revenue generated by each customer over all their orders.
```sql
SELECT customer_id,sum(total_amount) as total_Revenue FROM zomato_project.customers
left join orders as o
using(customer_id)
group by customer_id
```
---
![image](https://github.com/user-attachments/assets/baa4b123-4b2b-4166-8b80-e7a05b3ef49e)

---
🟢.Q17.Identify sales trends by comparing each month's total sales to the previous month.
```sql
with cte as 
(
SELECT year(order_date) as "yr",
monthname(order_date) as "month_name",
month(order_date) as "month_num", 
sum(total_amount) as "total_sales"

FROM orders group by 1,2,3 order by 1,3 
)
select *,lag(total_sales,1,0) over() as "prev_value",
round((
(total_Sales -lag(total_sales,1,0) over()) / lag(total_sales,1,0) over())*100,2)
as growth
from cte
```
---
![image](https://github.com/user-attachments/assets/8ce43107-ad1e-4694-ac73-ac0fbc419f8a)

---
🟢.Q18.Evaluate rider efficiency by determining average delivery times and identifying those with the lowest and highest averages(0nly top 2 and bottom 2)
```sql
with cte as (SELECT r.rider_id,d.order_id,o.order_time ,d.delivery_time  FROM riders r
left join  deliveries as d
using(rider_id)
join orders as o
using(order_id)
where d.delivery_status = "Delivered"
)
,
cte1 as (
select rider_id,order_id,
case 
	when order_time then ((extract(hour from order_time)*3600) + (extract(minute from order_time)*60) + (extract(second from order_time)))
    end order_time_in_minutes
,case 
	when delivery_time < order_time then ((extract(hour from delivery_time)+24)*3600 + extract(minute from delivery_time)*60 + extract(second from delivery_time))
    else (extract(hour from delivery_time)*3600 + extract(minute from delivery_time)*60 + extract(second from delivery_time))
   end delivery_time_in_minutes
from cte
),
cte2 as (select *, round((delivery_time_in_minutes - order_time_in_minutes)/60,2) as duration_delivery_time from cte1
),
cte3 as (
select rider_id,round(avg(duration_delivery_time),2) as avg_efficiency_time,"top_2" as avg_eff from cte2
group by rider_id
order by 2 desc limit 2
),
cte4 as (
select rider_id,round(avg(duration_delivery_time),2) as avg_efficiency_time,"bottom_2" as avg_eff from cte2
group by rider_id
order by 2 limit 2
)

select * from cte3 
 union
select * from cte4
```
---
![image](https://github.com/user-attachments/assets/c6c7eec4-6f8f-4369-a6ae-dfd39d444013)

---
🟢.Q19.Track the popularity of specific order items over time and identify seasonal demand spikes.
```sql
select order_item,
count(case when month(order_date) in(12,1,2) then 1 else null end) as "Winter",
count(case when month(order_date) in(3,4,5) then 1 else null end) as "Spring",
count(case when month(order_date) in(6,7,8) then 1 else null end) as "summer",
count(case when month(order_date) in(9,10,11) then 1 else null end) as "Autumn"
from orders
group by order_item
order by 2 desc,3 desc,4 desc,5 desc
```
---
![image](https://github.com/user-attachments/assets/4781ffd4-7af8-491b-a106-14027c3d260b)

---
🟢.Q20.Rank each city based on the total revenue for the last year (2023).
```sql
SELECT city,sum(total_amount) as total_revenue,
rank() over(order by sum(total_amount) desc) as rnk
FROM restaurants as r
left join orders as o
using(restaurant_id)
where year(order_date) = 2023
group by city
```
---
![image](https://github.com/user-attachments/assets/fcada926-e18b-4ee4-8207-d192ecd4e379)

---
