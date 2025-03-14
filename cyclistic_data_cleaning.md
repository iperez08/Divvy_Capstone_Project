## Create new table with merged data from all tables

```sql
CREATE TABLE `stellar-utility-451121-f7.cyclistic.past_year_trips` AS
SELECT * FROM `stellar-utility-451121-f7.cyclistic.february_24_trips` UNION ALL
SELECT * FROM `stellar-utility-451121-f7.cyclistic.march_24_trips` UNION ALL
SELECT * FROM `stellar-utility-451121-f7.cyclistic.april_24_trips` UNION ALL
SELECT * FROM `stellar-utility-451121-f7.cyclistic.may_24_trips` UNION ALL
SELECT * FROM `stellar-utility-451121-f7.cyclistic.june_24_trips` UNION ALL
SELECT * FROM `stellar-utility-451121-f7.cyclistic.july_24_trips` UNION ALL
SELECT * FROM `stellar-utility-451121-f7.cyclistic.august_24_trips` UNION ALL
SELECT * FROM `stellar-utility-451121-f7.cyclistic.september_24_trips` UNION ALL
SELECT * FROM `stellar-utility-451121-f7.cyclistic.october_24_trips` UNION ALL
SELECT * FROM `stellar-utility-451121-f7.cyclistic.november_24_trips` UNION ALL
SELECT * FROM `stellar-utility-451121-f7.cyclistic.december_24_trips` UNION ALL
SELECT * FROM `stellar-utility-451121-f7.cyclistic.january_25_trips`
```

## Explore Data

Check for nulls
```sql
SELECT 
  COUNTIF(rideable_type IS NULL) AS count_ride_type_nulls,
  COUNTIF(started_at IS NULL) AS count_start_at_nulls,
  COUNTIF(ended_at IS NULL) AS count_end_at_nulls,
  COUNTIF(start_station_name IS NULL) AS count_start_station_nulls,
  COUNTIF(end_station_name IS NULL) AS count_end_station_nulls,
  COUNTIF(start_station_id IS NULL) AS count_start_station_id_nulls,
  COUNTIF(end_station_id IS NULL) AS count_end_station_id_nulls,
  COUNTIF(start_lat IS NULL) AS count_start_lat_nulls,
  COUNTIF(start_lng IS NULL) AS count_start_lng_nulls,
  COUNTIF(end_lat IS NULL) AS count_end_lat_nulls,
  COUNTIF(end_lng IS NULL) AS count_end_lng_nulls,
  COUNTIF(member_casual IS NULL) AS count_member_casual_nulls
FROM `stellar-utility-451121-f7.cyclistic.past_year_trips`
```

Check for duplicates
```sql
SELECT 
  COUNT(ride_id) - COUNT(DISTINCT ride_id) AS count_duplicates
FROM `stellar-utility-451121-f7.cyclistic.past_year_trips`
```

Explore Duplicate Data
```sql
SELECT
*
FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
WHERE ride_id IN
  (SELECT 
  ride_id
  FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
  GROUP BY
    ride_id
  HAVING
    COUNT(ride_id) > 1)
ORDER BY
  ride_id
```

## Clean & Transform Data
1. Creates a trip_duration column in seconds and removes rows with nulls in start_station_name or end_station_name columns
```sql
CREATE OR REPLACE TABLE `stellar-utility-451121-f7.cyclistic.cleaned_past_trips` AS
SELECT 
  ride_id,
  rideable_type, 
  started_at,
  ended_at,
  start_station_name,
  end_station_name,
  TIMESTAMP_DIFF(ended_at, started_at, SECOND) AS trip_duration,
  member_casual
FROM `stellar-utility-451121-f7.cyclistic.past_year_trips`
WHERE start_station_name IS NOT NULL 
  AND end_station_name IS NOT NULL;
```

2. Created new columns: day_of_week, day, month, year, hour, season, and time_of_day and removed trip_durations that are 0 seconds or have a negative duration
```sql
CREATE OR REPLACE TABLE `stellar-utility-451121-f7.cyclistic.cleaned_all_past_trips`
SELECT
  *,
  FORMAT_TIMESTAMP('%A', started_at) AS day_of_week,
  EXTRACT(DAY FROM started_at) AS day,
  DATE_TRUNC(DATE(started_at), MONTH) AS month,
  EXTRACT(YEAR FROM started_at) AS year,
  EXTRACT(HOUR FROM started_at) AS hour,
  CASE EXTRACT(MONTH FROM started_at)
    WHEN 12, 1, 2 THEN 'Winter'
    WHEN 3, 4, 5 THEN 'Spring'
    WHEN 6, 7, 8 THEN 'Summer'
    ELSE 'Fall'
  END AS season,
  CASE 
    WHEN EXTRACT(HOUR FROM started_at) BETWEEN 6 AND 10 THEN 'Morning'
    WHEN EXTRACT(HOUR FROM started_at) BETWEEN 10 AND 14 THEN 'Midday'
    WHEN EXTRACT(HOUR FROM started_at) BETWEEN 14 AND 18 THEN 'Afternoon'
    WHEN EXTRACT(HOUR FROM started_at) BETWEEN 18 AND 22 THEN 'Evening'
    ELSE 'Night'
  END AS time_of_day
FROM `stellar-utility-451121-f7.cyclistic.cleaned_past_trips`
WHERE trip_duration > 0;
```

3. Remove all instances of duplicates and partitions table by month
```sql
CREATE OR REPLACE TABLE `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
PARTITION BY month AS
SELECT 
*
FROM
`stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
WHERE NOT (
    ride_id IN (
      SELECT ride_id
      FROM `stellar-utility-451121-f7.cyclistic.partitioned_all_past_trips`
      WHERE month = '2024-05-01'
      GROUP BY ride_id
      HAVING COUNT(*) > 1
    )
)
```

