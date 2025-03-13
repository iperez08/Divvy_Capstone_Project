## Big Picture Data

total number of trips, average length of trip, and max length of trip
```sql
SELECT 
  COALESCE(member_casual, 'total') AS riders,
  COUNT(*) AS trips_count,
  ROUND(AVG(trip_duration) / 60) AS avg_trip_duration_min,
  ROUND(MAX(trip_duration) / 60) AS max_trip_duration_min
FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
GROUP BY 
  GROUPING SETS ((member_casual), ())
ORDER BY 
  riders
```

Total Trips by Rider
```sql
SELECT 
member_casual,
COUNT(*) AS total_trips
FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
GROUP BY
  member_casual
```

Average Length of Trip by Rider
```sql
SELECT
  member_casual,
  ROUND(AVG(trip_duration) / 60) AS avg_trip_duration
FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
GROUP BY
  member_casual
```

Total Trips by Vehicle by Rider
```sql
SELECT
*
FROM (
  SELECT
    COALESCE(member_casual,'total') AS riders,
    rideable_type AS bike_type,
  COUNT(*) AS trip_count
  FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
  GROUP BY GROUPING SETS ((member_casual,rideable_type),(rideable_type))
)
PIVOT 
  (
    SUM(trip_count) FOR bike_type IN (
      'classic_bike', 'electric_bike', 'electric_scooter'
    )
  )
ORDER BY riders
```

Average Length of Trip by Vehicle by Rider
```sql
SELECT
*
FROM (
  SELECT
    COALESCE(member_casual,'total') AS riders,
    rideable_type AS bike_type,
    AVG(trip_duration) AS avg_trip_duration
  FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
  GROUP BY GROUPING SETS ((member_casual,rideable_type),(rideable_type))
)
PIVOT 
  (
    SUM(ROUND(avg_trip_duration / 60)) FOR bike_type IN (
      'classic_bike', 'electric_bike', 'electric_scooter'
    )
  )
ORDER BY riders
```

## Trends Across Time - Total Rides 

Total Trips by Month by Rider
```sql
SELECT *
FROM (
  SELECT
    COALESCE(member_casual, 'tota') AS riders,
    month,
    COUNT(*) as trip_count
    FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
    GROUP BY GROUPING SETS ((member_casual, month), (month))
)
PIVOT (
  SUM(trip_count) FOR month IN (
    "2024-02-01" AS `February_24`, "2024-03-01" AS `March_24`, 
    "2024-04-01" AS `April_24`, "2024-05-01" AS `May_24`,
    "2024-06-01" AS `June_24`, "2024-07-01" AS `July_24`,
    "2024-08-01" AS `August_24`, "2024-09-01" AS `September_24`,
    "2024-10-01" AS `October_24`, "2024-11-01" AS `November_24`,
    "2024-12-01" AS `December_24`, "2025-01-01" AS `January_25`
  )
)
ORDER BY
  riders
```

Total Trips by Day of Week by Rider
```sql
SELECT *
FROM (
  SELECT 
    COALESCE(member_casual, 'total') AS riders,
    day_of_week,
    COUNT(*) AS trip_count
  FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
  GROUP BY GROUPING SETS ((member_casual, day_of_week), (day_of_week))
)
PIVOT (
  SUM(trip_count) FOR day_of_week IN (
    "Monday", "Tuesday", "Wednesday", "Thursday",
    "Friday", "Saturday", "Sunday"
  )
)
ORDER BY 
  riders;
```

Total Trips by Hour by Rider
```sql
SELECT *
FROM (
  SELECT 
    COALESCE(member_casual, 'total') AS riders,
    hour,
    COUNT(*) AS trip_count
  FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
  GROUP BY GROUPING SETS ((member_casual, hour), (hour))
)
PIVOT (
  SUM(trip_count) FOR hour IN (
    0 AS `12am`, 1 AS `1am`, 2 AS `2am`, 3 AS `3am`, 4 AS `4am`, 5 AS `5am`,
    6 AS `6am`, 7 AS `7am`, 8 AS `8am`, 9 AS `9am`, 10 AS `10am`, 11 AS `11am`,
    12 AS `12pm`, 13 AS `1pm`, 14 AS `2pm`, 15 AS `3pm`, 16 AS `4pm`, 17 AS `5pm`,
    18 AS `6pm`, 19 AS `7pm`, 20 AS `8pm`, 21 AS `9pm`, 22 AS `10pm`, 23 AS `11pm`
  )
)
ORDER BY 
  riders;
```

## Trends Across Time - Average Length of Trips

Average Length of Trip by Month by Rider
```sql
SELECT
*
FROM (
  SELECT 
      COALESCE(member_casual, 'total') AS riders,
      month,
      ROUND(AVG(trip_duration) / 60,0) AS avg_trip_length
  FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
  GROUP BY GROUPING SETS ((member_casual, month), (month))
)
PIVOT (
  SUM(avg_trip_length) FOR month IN (
    "2024-02-01" AS `February_24`, "2024-03-01" AS `March_24`, 
    "2024-04-01" AS `April_24`, "2024-05-01" AS `May_24`,
    "2024-06-01" AS `June_24`, "2024-07-01" AS `July_24`,
    "2024-08-01" AS `August_24`, "2024-09-01" AS `September_24`,
    "2024-10-01" AS `October_24`, "2024-11-01" AS `November_24`,
    "2024-12-01" AS `December_24`, "2025-01-01" AS `January_25`
  )
)
ORDER BY
  riders
```

Average Length of Trip by Day of Week by Rider
```sql
SELECT
*
FROM (
  SELECT 
      COALESCE(member_casual, 'total') AS riders,
      day_of_week,
      ROUND(AVG(trip_duration) / 60,0) AS avg_trip_length
  FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
  GROUP BY GROUPING SETS ((member_casual, day_of_week), (day_of_week))
)
PIVOT (
  SUM(avg_trip_length) FOR day_of_week IN (
    "Monday", "Tuesday", "Wednesday", "Thursday",
    "Friday", "Saturday", "Sunday"
  )
)
ORDER BY
  riders
```

Average Length of Trip by Hour by Rider
```sql
SELECT
*
FROM (
  SELECT 
      COALESCE(member_casual, 'total') AS riders,
      hour,
      ROUND(AVG(trip_duration) / 60,0) AS avg_trip_length
  FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
  GROUP BY GROUPING SETS ((member_casual, hour), (hour))
)
PIVOT (
  SUM(avg_trip_length) FOR hour IN (
    0 AS `12am`, 1 AS `1am`, 2 AS `2am`, 3 AS `3am`, 4 AS `4am`, 5 AS `5am`,
    6 AS `6am`, 7 AS `7am`, 8 AS `8am`, 9 AS `9am`, 10 AS `10am`, 11 AS `11am`,
    12 AS `12pm`, 13 AS `1pm`, 14 AS `2pm`, 15 AS `3pm`, 16 AS `4pm`, 17 AS `5pm`,
    18 AS `6pm`, 19 AS `7pm`, 20 AS `8pm`, 21 AS `9pm`, 22 AS `10pm`, 23 AS `11pm`
  )
)
ORDER BY
  riders
```

## Trends Across Stations

Top Start Stations by Rider
```sql
WITH popular_start_station AS (
  SELECT
    COALESCE(member_casual, 'total') AS riders,
    start_station_name,
    COUNT(*) AS station_count,
  FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
  GROUP BY 
    GROUPING SETS ((member_casual, start_station_name), (start_station_name))
)
SELECT
  *
FROM (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY riders ORDER BY station_count DESC) AS row_num
  FROM popular_start_station
)
WHERE row_num IN (1, 2, 3, 4, 5)
```

Top End Stations by Rider
```sql
WITH popular_end_station AS (
  SELECT
    COALESCE(member_casual, 'total') AS riders,
    end_station_name,
    COUNT(*) AS station_count,
  FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
  GROUP BY 
    GROUPING SETS ((member_casual, end_station_name), (end_station_name))
)
SELECT
  *
FROM (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY riders ORDER BY station_count DESC) AS row_num
  FROM popular_end_station
)
WHERE row_num IN (1, 2, 3, 4, 5)
```

Top Routes by Rider
```sql
WITH popular_routes AS (
  SELECT
    COALESCE(member_casual, 'total') AS riders,
    CONCAT(start_station_name, ' to ', end_station_name) AS route,
    COUNT(*) AS route_count,
  FROM `stellar-utility-451121-f7.cyclistic.tableau_all_past_trips`
  GROUP BY 
    GROUPING SETS ((member_casual, route),(route))
)
SELECT
  *
FROM (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY riders ORDER BY route_count DESC) AS row_num
  FROM popular_routes
)
WHERE row_num IN (1, 2, 3, 4, 5)
```
