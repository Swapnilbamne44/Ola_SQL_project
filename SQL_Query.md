select * from trips;

select * from trips_details4;

select * from loc;

select * from duration;

select * from payment;


--total trips

select count(distinct tripid) as total_trips from trips;


--total drivers

select * from trips;

select count(distinct driverid) as Total_drivers from trips;

-- total earnings

select sum(fare) as total_earnings from trips;

-- total Completed trips

select * from trips_details4;
select sum(end_ride) as Completed_trips from trips_details4;

--total searches

select sum(searches) as searches from trips_details4; 

select sum(searches) as searches,sum(searches_got_estimate) as searches_got_estimate ,sum(searches_for_quotes) as searches_for_quotes ,sum(searches_got_quotes) as searches_got_quotes,
sum(customer_not_cancelled) as customer_not_cancelled ,sum(driver_not_cancelled) as driver_not_cancelled,sum(otp_entered) as otp_entered,sum(end_ride) as end_ride
from trips_details4;

--total searches which got estimate

select sum(searches_got_estimate) as total_searches_got_estimate from trips_details4; 

select * from trips;


select * from trips_details4;


--total driver cancelled

select count(*) - sum(driver_not_cancelled) as total_driver_cancelled from trips_details4;

--cancelled bookings by customer

select count(*) - sum(customer_not_cancelled) as total_customer_cancelled from trips_details4;

--average distance per trip

select sum(distance)/count(*) as Avg_distance from trips;

--average fare per trip

select sum(fare)/count(*) as Avg_fare from trips;

--distance travelled
select sum((distance)) as distance_travelled from trips;

-- which is the most used payment method 
select method from payment;

select top 1 count(distinct t.tripid) as cnt, p.method  from trips t
join payment p
on t.faremethod = p.id 
group by faremethod, method
order by count(distinct t.tripid) desc;

Select * from payment;
Select * from trips;
select * from loc;
select * from trips_details4;
select * from duration;

select m.method from payment m 
inner join
(select top 1 faremethod, count(distinct tripid) as cnt from trips
group by faremethod
order by count(distinct tripid) desc) b
on m.id = b.faremethod;


-- the highest payment was made through which instrument => creadit card
select m.method from payment m 
inner join
(select top 1 faremethod ,sum(fare) as fare from trips
group by faremethod
order by sum(fare) desc) as b
on m.id = b.faremethod;

select m.method from payment m 
inner join
(select top 1 faremethod, fare from trips
order by fare desc) b
on m.id = b.faremethod;

-- which two locations had the most trips
select * from loc;
select * from trips;

select sum(distinct loc_to) as locations from trips
group by loc_to
order by sum(loc_to) desc; 

select top 2 sum(distinct t.loc_to) as locations, l.assembly1 from trips t
join loc l
on l.id = t.loc_to
group by loc_to, assembly1
order by sum(loc_to) desc; 


select * from
(select *, dense_rank() over (order by trip desc) rnk
from
(select loc_from, loc_to, count(distinct tripid) as trip from trips
group by loc_from, loc_to) a )b
where rnk = 1;

select t.loc_from, l.assembly1 from trips t 
join loc l
on l.id = t.loc_from;


--top 5 earning drivers
select * from trips;

select top 5 driverid, sum(fare) as fare from trips
group by driverid
order by sum(fare) desc;

select * from
(select *, dense_rank() over (order by fare desc) rnk
from
(select driverid, sum(fare) as fare from trips
group by driverid) a)b
where rnk<=5;
 

-- which duration had more trips
select * from duration;

select * from
(select *, rank() over (order by cnt desc) rnk
from
(select duration, count(distinct tripid) as cnt from trips
group by duration) a)b
where rnk=1;




-- which driver , customer pair had more orders

select * from
(select *,rank() over (order by cnt desc) rnk
from
(select driverid, custid, count(distinct tripid) as cnt from trips
group by driverid, custid)a)b
where rnk=1;

-- search to estimate rate

select * from trips_details4;
select sum(searches_got_estimate) as search_to_estimate_rate from trips_details4;

select sum(searches_got_estimate) * 100.00/sum(searches) as search_to_estimate_rate from trips_details4;

-- estimate to search for quote rates
select sum(searches_for_quotes) * 100.00/sum(searches) as searches_for_quotes from trips_details4;

-- quote acceptance rate
select sum(searches_got_quotes) * 100.00/sum(searches) as searches_got_quotes from trips_details4;

-- quote to booking rate
select sum(otp_entered) * 100.00/sum(searches) as searches_got_quotes from trips_details4;


-- booking cancellation rate
select sum(searches) from trips_details4;
select sum(searches)-sum(customer_not_cancelled) as cancel_by_cust from trips_details4 
union 
select sum(searches)-sum(driver_not_cancelled) as cancel_by_driver from trips_details4;
-- select sum(otp_entered)*100.00/sum(searches) as booking_done from trips_details4;

-- conversion rate

select sum(otp_entered) * 100.00/sum(searches) as conversion_rate from trips_details4;

-- which area got highest trips in which duration;
select * from
(select*,rank() over (partition by duration order by cnt desc) rnk 
from
(select duration, loc_from, count(distinct tripid) as cnt from trips
group by duration, loc_from)a)b
where rnk = 1;

select * from
(select*,rank() over (partition by loc_from order by cnt desc) rnk 
from
(select duration, loc_from, count(distinct tripid) as cnt from trips
group by duration, loc_from)a)b
where rnk = 1;

-- which area got the highest fares, cancellations,trips,

select * from
(select *,rank() over (order by fare desc) rnk
from
(select loc_from, sum(fare) as fare from trips
group by loc_from)a)b
where rnk = 1; -- fare highest

select * from
(select*, rank() over (order by cancelled desc) rnk
from
(select loc_from, count(*) - sum(driver_not_cancelled) as cancelled from trips_details4
group by loc_from)a)b
where rnk = 1 -- cal by driver

select * from
(select*, rank() over (order by cancelled desc) rnk
from
(select loc_from, count(*) - sum(customer_not_cancelled) as cancelled from trips_details4
group by loc_from)a)b
where rnk = 1 -- cal by customer

-- which duration got the highest trips and fares

select * from
(select*,rank() over (order by fare desc) rnk 
from
(select duration, count(distinct tripid) as trip,sum(fare) as fare from trips
group by duration)a)b
where rnk = 1;
