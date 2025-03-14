# Divvy Ridership Data Analysis

## Overview
This project analyzes data to identify trends in ridership data among two kinds of riders: Divvy members and casual riders. By examining factors such as vehicle, month, day of the week, time of day, and stations, it provides insight into how Divvy members and casual riders use Divvy's services.

## Business Goal
Design a marketing campaign to convert casual riders into annual members.

## Key Features
- SQL-based data transformation and extraction from a large dataset.
- Python-based data extraction from JSON feed.
- Data visualizations created with Microsoft Excel to uncover patterns in user trends.
- Data visualizations created with Tableau to demonstrate key insights.

## Technologies and Skills
- **Technologies**: Big Query, SQL, Python, Tableau, GitHub.
- **Skills**: Data analysis, SQL query optimization, data cleaning, data visualization.

## Data Sources
Click [here](https://divvy-tripdata.s3.amazonaws.com/index.html) to access the ridership data from Divvy. Click [here](https://gbfs.lyft.com/gbfs/2.3/chi/en/station_information.json) to access the station information JSON feed.

## Processing Data
I used BigQuery to store, clean, and transform my data. All data from Divvy comes monthly downloads, so I used BigQuery to merge all of the tables from February 2024 through January 2025 into one new table.

### 1. Data Exploration
I checked for nulls among key columns and for duplicates in my newley created table. I was able to find 1.6 million rows that had no entries for start or end locations, and among them over 1 million of those entries came from electric bikes. This is an issue that Divvy may want to address with its fleet of bikes. I was also able to find rows that were duplicate data. This duplicate data was concentrated around trips that started on May 31st and ended on June 1st. Each trip was included in the May data file and the July data file.

### 2. Data Transformation & Cleaning
I created new columns for my data: day of the week, day, month, year, hour, season, time of day, as well as trip duration. I removed rows that had no start or end location as well as any rows that had trip durations of 0 seconds or less. Once I had this table, I then removed any duplicate data and partitioned the table by month in case I wanted to perform queries on specific months.

### 3. Tableau Data Transformation & Cleaning
I combined the queries I used for my original data cleaning and transformation, while also adjusting the query to include latitude and longitude columns to use in Tableau's map features and changing the trip_duration column to minutes rounded to the nearest hundredth. After loading the data in Tableau, I realized that every station had multiple latitude and longitude columns, which became an issue once I wanted to visualize stations on a map. I used Python to extract station information data from Divvy's JSON feeds and upload it as a BigQuery table. Joining the two tables on Tableau then allowed me to map popular stations using trusted and consistent data.

## Analyzing Data
I used BigQuery and Excel to analyze my data. I used SQL to query my database. Once I verified my query, I downloaded the table as a CSV file, and I used Excel to create quick charts.

### Overall Trends
I started by looking at the total number of trips, average length of trips, and the maximum length of trips. I was also able to see the total number of trips and the average length of trips for each customer type. I dove a little deeper to gain insight into how each customer type used the different vehicle options. The biggest takeaways were that casual members account for 36% of all rides, and average casual rides (24 min.) are almost twice as long as average Divvy member rides (12.4 min.). There were no significant differences in how each customer type used the different vehicle types.

### Trends Across Time - Total Trips and Average Length of Trips
I then looked at the trend across different time intervals, namely by month, by day of the week, and by hour in order to see if I could find deviations from total rides or average length of rides.

When we look at total trips by month, summer months have the highest rideship overall, including for casual riders. July is the peak ridership month for casual riders when they make up 43% of all rides.

When we look at total trips by days of the week, weekends are the highest ridership days for casual riders and the lowest ridership days for Divvy members. Casual riders make up 48% of all rides on weekends, a signficant deviation from the average 36% of all rides.

When we look at total trips by hour, 5pm has the highest ridership overall, including for casual riders. However, casual riders make up 33% of all rides. We can  look at 2pm when casual ridership makes up 43% and is the 5th busiest hour.

When we look at average length of trips by month, we can see that from March through October casual riders' trips are almost twice as long Divvy member rides, which is what we expect. It isn't until December and January when average casual rides fall below the overall average.

When we look at average length of trips by day of the week, we can see that casual riders' trips are slightly longer on the weekends at about 27.2 and 27.7 minutes for Saturday and Sunday, respectively.

Lastly, from 9am to 4pm, casual riders' average length of trips by hour are at least almost twice as long as Divvy members.

### Location Trends
When we look at the top 5 start stations, top 5 end stations, and top 5 routes for each kind of rider, we see interesting patterns. The top 5 start stations and top 5 end stations for casual riders are the same. This goes for Divvy members as well. Four out of the top 5 routes for casual riders are routes that start and end in the same station. However, for Divvy members, none of their top 5 stations show up in the top 5 routes. 
