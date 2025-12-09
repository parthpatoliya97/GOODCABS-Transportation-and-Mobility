# GOODCABS Operational Performance & Passenger Analysis


## 📌 Project Overview :

- Goodcabs is a fast-growing cab service company operating across ten tier-2 cities in India. Established two years ago, the company differentiates itself by empowering local drivers and delivering an excellent passenger experience. As part of its 2024 strategic growth plan, Goodcabs aims to evaluate key performance metrics across its transportation operations.

- This Power BI project provides actionable insights for the Chief of Operations to support data-driven decision-making around trip performance, driver efficiency, and passenger satisfaction.


## Problem Statement :

Goodcabs has set ambitious 2024 performance targets to expand its market presence in the Indian transportation and mobility sector. To support these goals, the Chief of Operations (Bruce Haryali) requires a detailed analysis of the company’s operational performance, 
focusing on:

- Trip volume trends

- Passenger satisfaction

- Repeat passenger behavior

- Trip distribution across segments

- Ratio of new vs. repeat passengers

- City-level performance and growth potential

-  Operational Performance KPIs

----------------------------------------

## Data Model View :

![goodcabs data model view](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/Data%20Model%20View.png?raw=true)

----------------------

#### Business Request - 1: City-Level Fare and Trip Summary Report
- Generate a report that displays the total trips, average fare per km, average fare per trip, and the percentage contribution of each city's trips to the overall trips. This report will help in assessing trip volume, pricing efficiency, and each city's contribution to the overall trip count.

```sql
WITH city_stats AS (
    SELECT
        c.city_name,
        COUNT(*) AS total_trips,
        SUM(t.distance_travelled_km) AS total_travelled_distance,
        SUM(t.fare_amount) AS total_revenue
    FROM dim_city c 
    JOIN fact_trips t 
        ON c.city_id = t.city_id
    GROUP BY c.city_name
)
SELECT 
    city_name,
    total_trips,
    ROUND(total_revenue / total_travelled_distance, 2) AS avg_fare_per_km,
    ROUND(total_revenue / total_trips, 2) AS avg_fare_per_trip,
    ROUND(total_trips*100 / (SELECT COUNT(*) FROM fact_trips), 2) AS contribution_to_total_trips
FROM city_stats;
```
![request 1](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/Business%20Request-1.png?raw=true)

------------------------------------

#### Business Request - 2: Monthly City-Level Trips Target Performance Report
- Generate a report that evaluates the target performance for trips at the monthly and city level. For each city and month, compare the actual total trips with the target trips and categorise the performance as follows:

- If actual trips are greater than target trips, mark it as "Above Target".

- If actual trips are less than or equal to target trips, mark it as "Below Target".

- Additionally, calculate the % difference between actual and target trips to quantify the performance gap.

```sql
WITH city_stats AS (
    SELECT
        MONTHNAME(t.date) AS month_name, 
        c.city_id,
        c.city_name,
        COUNT(*) AS total_trips
    FROM
        dim_city c
    JOIN
        fact_trips t ON c.city_id = t.city_id
    GROUP BY
        MONTHNAME(t.date),
        c.city_id,
        c.city_name
),
target_stats AS (
    SELECT
        MONTHNAME(target.month) AS month_name, 
        target.city_id,
        target.total_target_trips
    FROM
        targets_db.monthly_target_trips AS target
),
actual_vs_target AS (
    SELECT
        cts.city_name,
        cts.month_name, 
        cts.total_trips,
        ts.total_target_trips,
        (cts.total_trips - ts.total_target_trips) AS actual_difference_gap
    FROM
        city_stats cts
    JOIN
        target_stats ts
        ON cts.month_name = ts.month_name AND cts.city_id = ts.city_id 
)
SELECT
    city_name,
    month_name, 
    total_trips AS actual_trips,
    total_target_trips AS target_trips,
    actual_difference_gap,
    CASE
        WHEN total_trips > total_target_trips THEN 'Above_Target'
        ELSE 'Below_Target'
    END AS performance_status
FROM
    actual_vs_target
ORDER BY
    month_name; 
```
![january target vs actuals](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/January%20target_vs_actuals.png?raw=true)
![february target vs actuals](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/February_target_vs_actuals.png?raw=true)
![March target vs actuals](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/March_target_vs_actuals.png?raw=true)
![april target vs actuals](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/April_target_vs_actuals.png?raw=true)
![may target vs actuals](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/May_target_vs_actuals.png?raw=true)
![june target vs actuals](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/June_target_vs_actuals.png?raw=true)

-----------------------

#### Business Request - 3: City-Level Repeat Passenger Trip Frequency Report
- Generate a report that shows the percentage distribution of repeat passengers by the number of trips they have taken in each city. Calculate the percentage of repeat passengers who took 2 trips, 3 trips, and so on, up to 10 trips. Each column should represent a trip count category, displaying the percentage of repeat passengers who fall into that category out of the total repeat passengers for that city.

- Fields: city_name, 2-Trips, 3-Trips, 4-Trips, 5-Trips, 6-Trips, 7-Trips, 8-Trips, 9-Trips, 10-Trips

```sql
with trips as(
select 
	c.city_name,
    sum(rtp.repeat_passenger_count) as total_trips,
    sum(case when rtp.trip_count='2-Trips' then repeat_passenger_count end) as two_trips,
     sum(case when rtp.trip_count='3-Trips' then repeat_passenger_count end) as three_trips,
      sum(case when rtp.trip_count='4-Trips' then repeat_passenger_count end) as four_trips,
       sum(case when rtp.trip_count='5-Trips' then repeat_passenger_count end) as five_trips,
        sum(case when rtp.trip_count='6-Trips' then repeat_passenger_count end) as six_trips,
         sum(case when rtp.trip_count='7-Trips' then repeat_passenger_count end) as seven_trips,
          sum(case when rtp.trip_count='8-Trips' then repeat_passenger_count end) as eight_trips,
           sum(case when rtp.trip_count='9-Trips' then repeat_passenger_count end) as nine_trips,
            sum(case when rtp.trip_count='10-Trips' then repeat_passenger_count end) as ten_trips
	from dim_repeat_trip_distribution  rtp
	join dim_city c 
	on rtp.city_id=c.city_id
	group by c.city_name
)
select
	city_name,
	round(two_trips/total_trips*100,2) as Trip_2,
	round(three_trips/total_trips*100,2) as Trip_3,
	round(four_trips/total_trips*100,2) as Trip_4,
	round(five_trips/total_trips*100,2) as Trip_5,
	round(six_trips/total_trips*100,2) as Trip_6,
	round(seven_trips/total_trips*100,2) as Trip_7,
	round(eight_trips/total_trips*100,2) as Trip_8,
	round(nine_trips/total_trips*100,2) as Trip_9,
	round(ten_trips/total_trips*100,2) as Trip_10
from trips;
```
![business request 3](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/Business%20Request%203.png?raw=true)

---------------------------

#### Business Request - 4: Identify Cities with Highest and Lowest Total New Passengers
- Generate a report that calculates the total new passengers for each city and ranks them based on this value. Identify the top 3 cities with the highest number of new passengers as well as the bottom 3 cities with the lowest number of new passengers, categorising them as "Top 3" or "Bottom 3" accordingly.

```sql
select
	c.city_name,
	sum(s.total_passengers) as total_passengers,
	sum(s.new_passengers) as new_passengers,
	round(sum(s.new_passengers)/sum(s.total_passengers)*100,2) as new_passenger_pct
from
	fact_passenger_summary s 
join
	dim_city c on s.city_id=c.city_id
	group by
		c.city_name
	order by
		sum(s.new_passengers) asc
	limit 3;
```
![top 3](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/business%20reuqest%204%20top%203%20city.png?raw=true)
![bottom 3 ](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/business%20request%204%20bottom%203%20city.png?raw=true)

------------------------------------------

#### Business Request - 5: Identify Month with Highest Revenue for Each City
- Generate a report that identifies the month with the highest revenue for each city. For each city, display the month_name, the revenue amount for that month, and the percentage contribution of that month's revenue to the city's total revenue.

```sql
WITH monthly_revenue AS (
    -- Step 1: Calculate revenue per city, per month
    SELECT
        c.city_name,
        MONTHNAME(t.date) AS month_name,
        SUM(t.fare_amount) AS monthly_revenue
    FROM
        fact_trips t
    JOIN
        dim_city c ON t.city_id = c.city_id
    GROUP BY
        c.city_name,
        MONTHNAME(t.date) 
),
city_total_revenue AS (
    SELECT
        c.city_name,
        SUM(t.fare_amount) AS total_city_revenue
    FROM
        fact_trips t
    JOIN
        dim_city c ON t.city_id = c.city_id
    GROUP BY
        c.city_name 
),
revenue_ranking as(
SELECT
    mr.city_name,
    mr.month_name,
    mr.monthly_revenue AS revenue,
    ROUND(mr.monthly_revenue / ctr.total_city_revenue * 100, 2) AS revenue_share,
    dense_rank() over(partition by mr.city_name order by mr.monthly_revenue desc) as rnk
FROM
    monthly_revenue mr
JOIN
    city_total_revenue ctr ON mr.city_name = ctr.city_name
)
select 
city_name,
month_name,
revenue,
revenue_share
from revenue_ranking 
where rnk=1;

```
![request 5](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/business%20request%205.png?raw=true)

--------------------------------------

#### Business Request - 6: Repeat Passenger Rate Analysis
- Generate a report that calculates two metrics:

- Monthly Repeat Passenger Rate: Calculate the repeat passenger rate for each city and month by comparing the number of repeat passengers to the total passengers.

- City-wide Repeat Passenger Rate: Calculate the overall repeat passenger rate for each city, considering all passengers across months.

```sql
WITH monthly_stats AS (
    SELECT
        s.city_id,
        c.city_name,
        s.month,
        MONTHNAME(s.month) AS month_name,
        s.total_passengers,
        s.repeat_passengers,
        ROUND(s.repeat_passengers * 100.0 / s.total_passengers, 2) AS monthly_repeat_passenger_rate
    FROM
        fact_passenger_summary s
    JOIN
        dim_city c ON s.city_id = c.city_id
)
SELECT
    city_name,
    month_name AS month,
    total_passengers,
    repeat_passengers,
    monthly_repeat_passenger_rate,
    ROUND(
        SUM(repeat_passengers) OVER (PARTITION BY city_name) * 100.0 /
        SUM(total_passengers) OVER (PARTITION BY city_name),
    2) AS city_repeat_passenger_rate
FROM
    monthly_stats
ORDER BY
    city_name,
    month; 
```
![luck](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/Lucknow%20repeat%20passengers.png?raw=true)
![mysore](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/Mysore%20repeat%20passengers.png?raw=true)
![chandi](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/chandigarh%20repeat%20passengers.png?raw=true)
![coimb](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/coimbatore%20repeat%20passengers.png?raw=true)
![indore](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/indore%20repeat%20passengers.png?raw=true)
![jaipur](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/jaipur%20repeat%20passengers.png?raw=true)
![kochi](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/kochi%20repeat%20passengers.png?raw=true)
![surat](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/surat%20repeat%20passengers.png?raw=true)
![vadodara](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/vadodara%20repeat%20passengers.png?raw=true)
![visakha](https://github.com/parthpatoliya97/GOODCABS-Transportation-and-Mobility/blob/main/Images/visakhapatnam%20repeat%20passengers.png?raw=true)

------------------

1. Top and Bottom Performing Cities
Identify the top 3 and bottom 3 cities by total trips over the entire analysis period.

```sql
    SELECT
        c.city_name,
        COUNT(*) AS total_trips
    FROM dim_city c 
    JOIN fact_trips t 
        ON c.city_id = t.city_id
    GROUP BY c.city_name
    ORDER BY total_trips 
    LIMIT 3;
```








8. Highest and Lowest Repeat Passenger Rate (RPR%) by City and Month
By City: Analyse the Repeat Passenger Rate (RPR%) for each city across the six-month period. Identify the top 2 and bottom 2 cities based on their RPR% to determine which locations have the strongest and weakest rates.

By Month: Similarly, analyse the RPR% by month across all cities and identify the months with the highest and lowest repeat passenger rates.

```sql
select
c.city_name,
sum(s.total_passengers) as total_passengers,
sum(s.new_passengers) as new_passengers,
sum(s.repeat_passengers) as repeat_passengers,
round(sum(s.repeat_passengers)*100/sum(s.total_passengers),2) as repeat_contribution
from dim_city c 
join fact_passenger_summary s 
on c.city_id=s.city_id
group by c.city_name
order by repeat_contribution 
limit 3;


select 
monthname(d.start_of_month) as month_name,
sum(s.total_passengers) as total,
sum(s.new_passengers) as new_,
sum(s.repeat_passengers) as repeat_,
round(sum(s.repeat_passengers)*100/sum(s.total_passengers),2) repeat_contribution
from dim_date d 
join fact_passenger_summary s 
on monthname(d.start_of_month)=monthname(s.month)
group by monthname(d.start_of_month)
```


3. Average Ratings by City and Passenger Type
Calculate the average passenger and driver ratings for each city, segmented by passenger type (new vs. repeat). Identify cities with the highest and lowest average ratings.
```sql
select
c.city_name,
round(avg(case when t.passenger_type='new' then passenger_rating end),1) new_passenger_rating,
round(avg(case when t.passenger_type='new' then driver_rating end),1) new_driver_rating,
round(avg(case when t.passenger_type='repeated' then passenger_rating end),1) repeat_passenger_rating,
round(avg(case when t.passenger_type='repeated' then driver_rating end),1) repeat_driver_rating
from fact_trips t 
join dim_city c 
on t.city_id=c.city_id
group by c.city_name;
```

```sql
select
c.city_name,
round(sum(t.fare_amount)/1000000,2) as revenue,
round(avg(t.fare_amount),2) as avg_fare_per_trip
from fact_trips t 
join dim_city c 
on t.city_id=c.city_id
group by c.city_name
order by avg_fare_per_trip desc;
```

```sql
select
c.city_name,
d.day_type,
round(sum(t.fare_amount),2) as revenue,
round(avg(t.fare_amount),2) as avg_fare_per_trip
from fact_trips t 
join dim_city c 
on t.city_id=c.city_id
join dim_date d 
on t.date=d.start_of_month
group by c.city_name,d.day_type
order by c.city_name;
```


```sql
select 
   d.day_type,
    ROUND(SUM(t.fare_amount) / 1000000, 2) AS revenue_in_millions,
    ROUND(AVG(t.fare_amount), 2) AS avg_fare_per_trip,
	ROUND(SUM(t.fare_amount)/(select sum(fare_amount) from fact_trips)*100,2) as revenue_contribution_pct
from fact_trips t 
join dim_date d 
on t.date=d.start_of_month
group by d.day_type;
```

```sql
select
d.day_type,
round(sum(s.total_passengers)/1000000,2) total_passengers,
round(sum(s.new_passengers)/1000000,2) new_passengers,
round(sum(s.repeat_passengers)/1000000,2) repeat_passengers
from dim_date d 
join fact_passenger_summary s 
on d.start_of_month=s.month
group by d.day_type;
```
