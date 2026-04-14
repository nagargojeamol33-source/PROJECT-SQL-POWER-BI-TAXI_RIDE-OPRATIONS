create database urban_ride;
use urban_ride;

-- 1. Understanding data types of columns
desc nyc_ride;

#22 columns n 53000 rows in table urban_ride with missing values
select count(*) from nyc_ride;  #34818

#just shows 10 rows bcz of large dataset
select * from nyc_ride ;

set sql_safe_updates=0;

-- 2. Changing column data types of pickup_datetime and dropoff_datetime columns
start transaction;
update nyc_ride set pickup_datetime=str_to_date(pickup_datetime,'%d-%m-%Y %H:%i'); #convert text to datetime
alter table nyc_ride modify pickup_datetime datetime;

select * from nyc_ride;
rollback;


update nyc_ride set dropoff_datetime=str_to_date(dropoff_datetime,'%d-%m-%Y %H:%i'); #convert text to datetime
alter table nyc_ride modify dropoff_datetime datetime;
desc nyc_ride;

-- 3. filter all empty strings(missing values)in a text columns
select count(*) as total_rows from nyc_ride where pickup_location=''; #8759 rows
select * from nyc_ride where pickup_location='';

select count(*) as total_rows from nyc_ride where dropoff_location=''; #8733 rows
select * from nyc_ride where dropoff_location='';

#filter all fake missing values in a text columns
select count(*) as total_rows from nyc_ride where payment_type='Unknown'; #8667 rows
select * from nyc_ride where payment_type='Unknown';

-- 4. missing values as numerical columns(as per requirements)
#missing values count using sum() function in numerical columns
select 
sum(trip_id is null) as trip_id_nulls, 
sum(pickup_datetime is null) as pickup_datetime_nulls,
sum(dropoff_datetime is null) as dropoff_datetime_nulls,
sum(passenger_count is null) as passenger_count_nulls,
sum(trip_distance is null) as trip_distance_nulls,
sum(fare_amount is null) as fare_amount_nulls,
sum(tip_amount is null) as tip_amount_nulls,
sum(tolls_amount is null) as tolls_amount_nulls,
sum(total_amount is null) as total_amount_nulls,
#sum(payment_type is null) as payment_type_nulls,
sum(vendor_id is null) as vendor_id_nulls,
#sum(rate_code is null) as rate_code_nulls,
#sum(pickup_location is null) as pickup_location_nulls,
#sum(dropoff_location is null) as dropoff_location_nulls,
#sum(trip_type is null) as trip_type_nulls,
sum(driver_rating is null) as driver_rating_nulls,
#sum(vehicle_type is null) as vehicle_type_nulls,
#sum(weather is null) as weather_nulls,
#sum(traffic_level is null) as traffic_level_nulls,
sum(surge_multiplier is null) as surge_multiplier_nulls,
#sum(booking_source is null) as booking_source_nulls,
sum(customer_id is null) as customer_id_nulls from nyc_ride;

#permanent update and delete missing values
#set sql_safe_updates=0;

-- 5. replace/update  empty/unknown values with NA values in a text columns 
start transaction;
update nyc_ride set pickup_location=null where trim(pickup_location)='';
update nyc_ride set dropoff_location=null where trim(dropoff_location)='';
update nyc_ride set payment_type=null where trim(payment_type)='Unknown';
select * from nyc_ride ;

#SELECT @@SQL_SAFE_UPDATES;

-- 6. %criteria for null values in text columns
select 
#  pickup_location 
sum(case 
    when pickup_location is null or trim(pickup_location)='' 
    #or pickup_location in ('Unknown','N/A','NA') 
    then 1 else 0 end)*100.0/count(*) as pickup_location_missing_percent,
# dropoff_location 
sum(case 
    when dropoff_location is null or TRIM(dropoff_location)='' 
    #or dropoff_location in ('Unknown','N/A','NA') 
    then 1 else 0 end)*100.0/count(*) as dropoff_location_missing_percent,
#  payment_type 
sum(case 
    when payment_type is null 
    or trim(payment_type)= '' or payment_type='Unknown'
    then 1 else 0 end)*100.0/count(*) as payment_type_missing_percent
from nyc_ride;

#pickup_location_missing_percent  => 25.15653
#dropoff_location_missing_percent => 25.08185
#payment_type_missing_percent     => 24.89230

-- 7. % criteria of NA values in all above 3 columns is between 5 to 30% so fill values with mode

#select pickup_location from nyc_ride where pickup_location is not null group by pickup_location order by count(*) desc limit 1; #Manhattan
#select dropoff_location from nyc_ride where dropoff_location is not null group by dropoff_location order by count(*) desc limit 1; #Queens
#select payment_type from nyc_ride where payment_type is not null group by payment_type order by count(*) desc limit 1; #UPI

start transaction;
 update nyc_ride set pickup_location=(select pickup_location from (select pickup_location from nyc_ride where pickup_location is not null group by pickup_location
order by count(*) desc limit 1)as temp )where pickup_location is null;  #23001 rows updated
select * from nyc_ride ;
 
 update nyc_ride  set dropoff_location=(select dropoff_location from (select dropoff_location from nyc_ride where dropoff_location is not null group by dropoff_location
order by count(*) desc limit 1)as temp)where dropoff_location is null;
select * from nyc_ride ;

update nyc_ride set payment_type=(select payment_type from (select payment_type from nyc_ride where payment_type is not null group by payment_type
order by count(*) desc limit 1)as temp)where payment_type is null;
select * from nyc_ride ;

rollback;

-- 8. Delete NA rows in a columns

#find Duplicate records from our table
select trip_id,count(*) from nyc_ride group by trip_id having count(*)>1;

#remove duplicates  records
start transaction;
delete from nyc_ride where trip_id in (select trip_id from(select trip_id,row_number() over (partition by trip_id order by trip_id) as row_no from nyc_ride)nr
where row_no>1);  #deleted 3984 rows
rollback;
#set sql_safe_updates=0;
select count(*) from nyc_ride ; #30834 rows got 

-- 9. Handle Invalid data(column values)
#remove negative fare
select count(fare_amount)from nyc_ride where fare_amount<0;  #675 rows got
select (count(fare_amount)*100.0)/(select count(*) from nyc_ride) as invalid_percent from nyc_ride where fare_amount<0;  #2.18914 % invalid value rates
start transaction;
update nyc_ride set fare_amount=abs(fare_amount) where fare_amount<0;
select * from nyc_ride;
rollback;

-- 10. duplicate records from fixed set of values
select distinct(payment_type) from nyc_ride;
select distinct(rate_code) from nyc_ride;
select distinct(pickup_location) from nyc_ride;
select distinct(dropoff_location) from nyc_ride;
select distinct(trip_type) from nyc_ride;
select distinct(vehicle_type) from nyc_ride;
select distinct(weather) from nyc_ride;
select distinct(traffic_level) from nyc_ride;
select distinct(booking_source) from nyc_ride;

-- 11.  Feature Engineering
#Trip Duration
alter table nyc_ride add trip_duration int;
update nyc_ride set trip_duration=timestampdiff(minute,pickup_datetime,dropoff_datetime); #convert into minutes
select * from nyc_ride;

# Extract pickup_date only
alter table nyc_ride add column pickup_date date;
update nyc_ride set pickup_date=date(pickup_datetime); #extract only pickup date from pickup_datetime column

# Extract month only
alter table nyc_ride add column pickup_month date;
alter table nyc_ride modify column pickup_month text; #chande data type of pickup_month column bcz we dont apply month() fun on date column
update nyc_ride set pickup_month=monthname(pickup_datetime); #extract only pickup month from pickup_datetime column

#Fare(Revenue) Per km
alter table nyc_ride add column fare_amt_per_km float;
update nyc_ride set fare_amt_per_km=fare_amount/trip_distance;

#pickup hour
alter table nyc_ride add column pickup_hour int;
update nyc_ride set pickup_hour=hour(pickup_datetime);
select * from nyc_ride;  #30834  records

#Day of Week
alter table nyc_ride add column day_of_week text;
update nyc_ride set day_of_week=dayname(pickup_date);  #for weekday/weekend analysis/trends

-- Validate Cleaned Data 
select count(*) from nyc_ride;  #30834  records
select min(fare_amount),max(fare_amount) from nyc_ride; #min=5.01,max= 199.98

-- 12. Analysis
#peak hours
select pickup_hour,count(*) as total_trips from nyc_ride group by pickup_hour order by total_trips desc;  #for when demand is highest

#Profit
#fare per km 

#peak hours
select pickup_hour,count(*) as total_trips from nyc_ride group by pickup_hour order by total_trips desc;  #for when demand is highest

#daily Trends
select pickup_date,count(*) as trips from nyc_ride group by pickup_date order by pickup_date desc;  #daily trip counts

#revenue analysis
select sum(total_amount)as total_revenue from nyc_ride;
select * from nyc_ride;

#Payment Method Analysis
select payment_type,count(*)trips from nyc_ride group by payment_type order by trips desc;

#Distance vs Fare
select avg(trip_distance) as avg_distance,avg(fare_amount) as avg_fare_amt from nyc_ride;

#Top 10 Highest Fare Trips 
select * from nyc_ride order by fare_amount desc limit 10;

#Avg trip_duration by hour 
select pickup_hour,avg(trip_duration) as avg_trip_duration from nyc_ride group by pickup_hour;

#revenue by day
select pickup_date,sum(total_amount) as total_revenue from nyc_ride group by pickup_date;

#Weather Impact
select weather,avg(fare_amount) as avg_fare_amt from nyc_ride group by weather;

#Traffic Impact
select traffic_level,avg(trip_duration) as avg_trip_duration from nyc_ride group by traffic_level;

#Revenue by Booking Source
select booking_source,sum(total_amount) as total_revenue from nyc_ride group by booking_source;

#Vehicle Performance
select vehicle_type,avg(fare_amount) as avg_fare_amt from nyc_ride group by vehicle_type;

#Repeated Customers
select distinct(customer_id) from nyc_ride;
select customer_id,sum(total_amount) as total_revenue from nyc_ride group by customer_id order by total_revenue;
select * from nyc_ride;

# Top High fare tips
select *  from nyc_ride order by fare_amount limit 10;

-- 13. Insights
# peak hours are 6-9pm ->high demand
# cash vs Card Usage pattern
# Longer trips genarate more revenue
# some trips are abnormal fares
# Peak demand occurs during evening hours → high customer demand
# Revenue is concentrated in specific time periods
# High traffic increases trip duration significantly
# Rainy weather leads to higher fares
# Certain vehicle types generate better revenue efficiency
# Mobile app bookings contribute the highest revenue
# Digital payments are more frequently used than cash


#visuals
# trips by hour
#Revenue over time 
#payment Type Distribution
#Distance vs Fare
