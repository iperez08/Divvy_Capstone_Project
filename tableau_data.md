# Tableau Data Preparation

## Merged Table Data Cleaning

```sql
-- creates table with filtered original columns,
-- helper columns (day of week, day, month, year, hour, season, and time of day)
-- trip duration is stored in minutes rounded to the nearest hundredth
-- and partitions the table by month.

CREATE OR REPLACE TABLE `stellar-utility-451121-f7.cyclistic.tableau_all_past_trips` AS
SELECT
  ride_id,
  rideable_type,
  started_at,
  ended_at,
  start_station_name,
  end_station_name,
  start_lat,
  start_lng,
  end_lat,
  end_lng,
  trip_duration,
  FORMAT_TIMESTAMP('%A', started_at) AS day_of_week,
  EXTRACT(DAY FROM started_at) AS day,
  FORMAT_DATETIME("%B", DATE(started_at)) AS month,
  EXTRACT(YEAR FROM started_at) AS year,
  EXTRACT(HOUR FROM started_at) AS hour,
  member_casual
FROM
  (SELECT
  *,
  CASE
    WHEN TIMESTAMP_DIFF(ended_at, started_at, SECOND) < 0
    THEN ROUND((TIMESTAMP_DIFF(ended_at, started_at, SECOND) + 86400) / 60, 2)
    ELSE ROUND(TIMESTAMP_DIFF(ended_at, started_at, SECOND) / 60, 2)
    END AS trip_duration
  FROM `stellar-utility-451121-f7.cyclistic.past_year_trips`
  WHERE
  start_station_name IS NOT NULL AND
  end_station_name IS NOT NULL
  )
WHERE
  trip_duration <> 0
```

## Latitude and Longitude Data Exploration

```sql
-- Inspects how many different latitudes and longitudes there are for each station.
SELECT 
  start_station_name,
  COUNT(DISTINCT start_lat) AS start_lat_count,
  COUNT(DISTINCT start_lng) AS start_lng_count,
FROM `stellar-utility-451121-f7.cyclistic.tableau_all_past_trips`
GROUP BY
 start_station_name
ORDER BY
  start_lat_count DESC
```

## Grab Station Information Data with Latitude and Longitude

1. Download JSON file and Upload to Google Cloud Storage
```python
import requests
from google.cloud import storage

json_url = "https://gbfs.lyft.com/gbfs/2.3/chi/en/station_information.json"
response = requests.get(json_url)

with open("station_information.json", "wb") as file:
    file.write(response.content)

print("JSON file downloaded successfully.")

def upload_to_gcs(bucket_name, source_file_name, destination_blob_name):
    storage_client = storage.Client(project="stellar-utility-451121-f7")
    bucket = storage_client.bucket(bucket_name)
    blob = bucket.blob(destination_blob_name)
    
    blob.upload_from_filename(source_file_name)

    print(
        f"File {source_file_name} uploaded to {destination_blob_name}."
    )

bucket_name = "ismael_p_cyclistic_datasets"
upload_to_gcs(bucket_name, "station_information.json", "station_information.json")
```

2. Load JSON into Big Query and Verify Data
```python
from google.cloud import bigquery

client = bigquery.Client(project='stellar-utility-451121-f7')

def load_json_to_bigquery(dataset_id, table_id, gcs_uri):
    table_ref = client.dataset(dataset_id).table(table_id)
    job_config = bigquery.LoadJobConfig(
        source_format=bigquery.SourceFormat.NEWLINE_DELIMITED_JSON,
        autodetect=True  # Let BigQuery infer schema
    )

    load_job = client.load_table_from_uri(
        gcs_uri, table_ref, job_config=job_config
    )
    load_job.result()  # Wait for the job to complete

    print(f"Loaded {load_job.output_rows} rows into {dataset_id}.{table_id}")

dataset_id = "cyclistic"
table_id = "station_information"
gcs_uri = f"gs://ismael_p_cyclistic_datasets/station_information.json"

load_json_to_bigquery(dataset_id, table_id, gcs_uri)

def query_bigquery(dataset_id, table_id):
    query = f"SELECT * FROM `{client.project}.{dataset_id}.{table_id}` LIMIT 10"
    
    results = client.query(query)
    for row in results:
        print(row)

query_bigquery('cyclistic', 'station_information')
```
