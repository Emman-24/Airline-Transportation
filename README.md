
# Airline Transportation Analysis SQL Project

**Project Title**: Airline Transportation  
**Database**: `air_transportation.sql`

Analyzing airline, trip, and passenger data using SQL to uncover insights for improving operations and passenger experiences.




## Objectives

1. **Updating Passenger Trip Dates**: Alter the passenger's trip table to adjust date data types and synchronize trip dates with departure times to ensure data consistency.
2. **Identifying Key Passengers**: Identify the names of passengers who frequently fly with specific airline companies.
3. **Route Performance Analysis**: Analyze flight routes by calculating average flight duration, total passengers, and income generated per route.
4. **Boening vs Airbus Comparasion**: Compare flight durations and frequencies between Boeing and Airbus aircraft types.
5. **Top Routes by Duration for Each Company**: Identify the top two routes with the longest average total flight durations for each airline company.
6. **ABC Passenger Segmentation**: Utilize ABC testing to segment passengers based on their contribution to total income, categorizing them as A, B, or C based on their cumulative share percentage
## Project Structure

### 1. Updating Passenger Trip Dates

This stage helps you clean up dates and match trip dates with departure times, making the data neat and ready for analysis as you move forward.

```sql
ALTER TABLE Pass_in_trip MODIFY COLUMN trip_date DATE;

UPDATE Pass_in_trip SET trip_date = CAST(trip_date AS DATE);

SELECT * FROM Pass_in_trip;
```
## 2.Identifying Key Passengers
Identify passengers who frequently fly with specific airline companies.By analyzing database records, i'll pinpoint passengers who have flown multiple times with particular airlines. This insight aids airlines in tailoring services to meet the needs of loyal customers.

```sql
SELECT p.passenger_name, COUNT(pit.trip_no) as num_flights,ac.company_name
FROM Passenger p
JOIN Pass_in_trip pit ON p.ID_psg = pit.ID_psg
JOIN Trip t ON pit.trip_no = t.trip_no
JOIN Airline_company ac ON t.ID_comp = ac.ID_comp
group by p.passenger_name,ac.company_name
HAVING COUNT(pit.trip_no) > 1
ORDER BY num_flights DESC;
```


## Route Performance Analysis

I need to examine the efficiency of flight routes by computing key metrics such as average flight duration, total passengers served, and income generated per route. This objective focuses on synthesizing data from various sources to derive insights into route profitability and passenger demand, helping optimize airline operations and resource allocation.

```sql
SELECT CONCAT(t.town_from, '-',t.town_to) AS route,
      AVG(TIMESTAMPDIFF(MINUTE, t.time_out, t.time_in)) AS avg_flight_duration,
      COUNT(pit.seat_number) as total_passengers,
      AVG(TIMESTAMPDIFF(SECOND, time_out, time_in)) * COUNT(pit.seat_number) * 0.01 as total_income
FROM Trip t
JOIN Pass_in_trip pit ON t.trip_no = pit.trip_no
GROUP BY route
ORDER BY total_income DESC;
```


## Boening vs Airbus Comparasion

I want to compare the average flight durations between Boeing and Airbus aircraft to see if there's a notable difference. Before doing that, I'll gather data on flight durations and frequencies for both Boeing and Airbus planes across different airline companies. I aim to collect information on how long these planes fly on average and how often they're used for flights.

```sql
SELECT 
    CASE 
        WHEN plane_type LIKE 'Boeing%' THEN 'Boeing'
        WHEN plane_type LIKE 'Airbus%' THEN 'Airbus'
    END AS aircraft_type,
    AVG(TIMESTAMPDIFF(MINUTE, time_out, time_in)) AS avg_flight_duration,
    COUNT(*) AS num_flights
FROM Trip
WHERE plane_type LIKE 'Boeing%' 
    OR plane_type LIKE 'Airbus%'
GROUP BY 
    CASE 
        WHEN plane_type LIKE 'Boeing%' THEN 'Boeing'
        WHEN plane_type LIKE 'Airbus%' THEN 'Airbus'
    END;
```
## Top Routes by Duration for Each Company
My goal is to pinpoint the top two routes with the lengthiest average flight durations for every airline company. This involves analyzing flight data to determine the average duration between departure and arrival cities. The query will organize the data by company, departure city, and arrival city, calculating the average duration for each route. Finally, it ranks the routes by duration, selecting only the top two routes for each company.

```sql
WITH RouteDuration AS (
    SELECT
        ac.company_name,
        t.town_from AS departure_city,
        t.town_to AS arrival_city,
        AVG(TIMESTAMPDIFF(MINUTE, time_out, time_in)) AS avg_flight_duration
    FROM Trip t
    JOIN Airline_company ac ON t.ID_comp = ac.ID_comp
    GROUP BY ac.company_name, t.town_from, t.town_to
),
RouteRanting AS (
    SELECT *,
        ROW_NUMBER() OVER (PARTITION BY company_name ORDER BY avg_flight_duration DESC) AS rating
    FROM RouteDuration
)
SELECT
    company_name,
    departure_city,
    arrival_city,
    avg_flight_duration
FROM RouteRanting WHERE rating <= 2;
```
## ABC Passenger Segmentation
The objective is to identify these high-contributing passengers by categorizing them as A, B, or C based on their cumulative share percentage of total income. By targeting these passengers, airlines can optimize marketing strategies and loyalty programs to enhance revenue.

```sql
WITH PassengerIncome AS (
    SELECT 
        p.ID_psg,
        p.passenger_name,
        SUM(TIMESTAMPDIFF(SECOND, t.time_out, t.time_in) * 0.01) as passenger_income_dollars
    FROM Passenger p
    JOIN Pass_in_trip pit ON p.ID_psg = pit.ID_psg
    JOIN Trip t ON pit.trip_no = t.trip_no
    GROUP BY p.ID_psg, p.passenger_name
),
TotalIncomeAndRanking AS (
    SELECT 
        ID_psg,
        passenger_name,
        passenger_income_dollars,
        passenger_income_dollars / SUM(passenger_income_dollars) OVER () * 100 as share_percent,
        SUM(passenger_income_dollars) OVER (ORDER BY passenger_income_dollars DESC) / 
        SUM(passenger_income_dollars) OVER () * 100 as cumulative_share_percent
    FROM PassengerIncome
)
SELECT 
    ID_psg,
    passenger_name,
    ROUND(passenger_income_dollars, 2) as passenger_income_dollars,
    ROUND(cumulative_share_percent, 2) as cumulative_share_percent,
    CASE 
        WHEN cumulative_share_percent <= 80 THEN 'A'
        WHEN cumulative_share_percent <= 95 THEN 'B'
        ELSE 'C'
    END as category
FROM TotalIncomeAndRanking
ORDER BY passenger_income_dollars DESC;
```


Here's the link to the project: https://hyperskill.org/projects/447

Check out my profile: https://hyperskill.org/profile/319072528
