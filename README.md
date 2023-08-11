# google-bikeshare-analysis

## Data Cleaning

### Lat / Long Data

Check if any start coordinate is null:
```
SELECT COUNT(*)
FROM bike_data
WHERE start_lat IS NULL
OR start_lng IS NULL
```

-> 0

Checking if any end coordinate is null:
```
SELECT COUNT(*)
FROM bike_data
WHERE end_lat IS NULL
OR end_lng IS NULL
```

-> 5973

While no row is missing its start latitude or longitude, 5973 rows are missing end latitude or longitude. However, since this is a very small percent of the overall data (5973 / 5859061 = 0.00101945), it’s probably safe to delete the missing entries. I'll use this for calcuating ride distance.

```
DELETE FROM bike_data
WHERE end_lat IS NULL
OR end_lng IS NULL
```

### Stations Data

While the lat / long data is what I'll need for distance, knowing the popular stations will help us know where we might target ads for casual riders.
```
SELECT COUNT(*)
FROM bike_data
WHERE end_station_id IS NULL
OR start_station_id IS NULL
```
-> 1319089

Stations with a missing start or end station ID account for roughly 23% of the total data, so we cannot delete them.

I will manually take a look at the list of unique stations to see if there are any obvious mistakes. One thing I noticed is a lot of stations that begin with "Public Rack".
```
SELECT COUNT(*)
FROM bike_data
WHERE start_station_name LIKE "Public Rack%"
```
-> 16622

Do these stations that start with “Public Rack -“ differ from stations that contain just the street name?

Example:
start_station_name | count(*)
--- | ---
Kenton Ave & Palmer St | 105
Public Rack - Kenton Ave & Palmer St | 176

I don't know of a way to find the answer in this case. For this analysis however, it is probably safe to just assume they’re different stations.


### Date Data

Check if there are null starting dates:
```
SELECT  COUNT(*)
FROM bike_data
WHERE started_at IS NULL
```
-> 0

Make sure the data falls within the expected range:
```
SELECT  COUNT(*)
FROM bike_data
WHERE started_at BETWEEN "2022-05-01 00:00:00" AND "2023-04-30 23:59:59"
```
-> 5853088

Check any rides beyond the range:
```
SELECT  COUNT(*)
FROM bike_data
WHERE started_at > "2023-04-30 23:59:59"
```
-> 0

Check any rides before the range:
```
SELECT  COUNT(*)
FROM bike_data
WHERE started_at < "2022-05-01 00:00:00"
```
-> 0

There are no null started_at values and all started_at values are within the timeframe.

```
SELECT COUNT(*)
FROM bike_data
WHERE ended_at IS NULL
```
-> 0

```
SELECT  COUNT(*)
FROM bike_data
WHERE ended_at NOT BETWEEN "2022-05-01 00:00:00" AND "2023-04-30 23:59:59"
```
->16

16 trips ended outside of the timeframe, but since all start times were before the end of April 2023, we can assume they started before midnight and finished after midnight 2023-04-30.

Checking the data confirms that these were nighttime trips starting 2023-04-30.

started_at | ended_at
--- | ---
2023-04-30 23:33:28 | 2023-05-01 00:08:42
2023-04-30 23:43:20 | 2023-05-01 00:02:14
2023-04-30 07:21:29 | 2023-05-01 07:11:32
2023-04-30 16:52:28 | 2023-05-01 00:16:44
2023-04-30 23:51:51 | 2023-05-01 00:06:38
2023-04-30 23:52:16 | 2023-05-01 00:06:50
2023-04-30 23:49:26 | 2023-05-01 00:21:55
2023-04-30 23:48:13 | 2023-05-01 00:05:04
2023-04-30 23:49:29 | 2023-05-01 00:01:43
2023-04-30 23:46:41 | 2023-05-01 00:31:22
2023-04-30 21:18:48 | 2023-05-01 08:06:56
2023-04-30 23:49:13 | 2023-05-01 00:03:23
2023-04-30 23:58:48 | 2023-05-01 00:08:08
2023-04-30 23:59:05 | 2023-05-01 00:03:17
2023-04-30 23:26:57 | 2023-05-01 00:05:49
2023-04-30 23:54:21 | 2023-05-01 00:06:35



103 rides have a later start time than end time, which doesn’t make sense, so they should be deleted: 
```
SELECT COUNT(*)
FROM bike_data
WHERE started_at > ended_at
```
-> 103

```
DELETE FROM bike_data
WHERE started_at > ended_at
```

The top 11 longest rides are significantly longer than the rest. Being such big outliers, I will delete them.

```
SELECT ride_id,  ROUND((julianday(ended_at) - julianday(started_at)) * 24, 2) AS time_difference_hours
FROM bike_data
ORDER BY time_difference_hours DESC
LIMIT 20
```
ride_id | time_difference_hours
--- | ---
DC510E6F98003A94 | 533.92
E5886B2D636415DF | 180.12
84C8FD571931B767 | 178.72
17405F31D17313B2 | 166.04
B139FE7DF42819B0 | 137.4
23B687E80DE52ED8 | 111.21
C999C6BBBEC63568 | 80.81
4464B4342DAC2E9D | 45.33
BE795EF38EF6001E | 44.15
D88D5192DF6A4536 | 39.16
B5DDACE4B9B2EDE6 | 38.09
301B824D3B4F34BD | 25.0
DE3EC4018C01D81D | 25.0
E07785A68608AB1B | 25.0
F1320411474D6868 | 25.0
6F88887A3B385693 | 25.0
7CBAC35F55622E12 | 25.0
32C0B6B7FD53567C | 25.0
0D199461EEFA8C72 | 25.0
89354D5720905EE2 | 25.0

```
DELETE FROM bike_data
WHERE ROUND((julianday(ended_at) - julianday(started_at)) * 24) > 25.9
```


Checking the shortest rides, it seems many rides have the same start time as end time. We should check this.
```
SELECT COUNT(*)
FROM bike_data
WHERE started_at = ended_at
```
-> 441

These trips likely have an incorrect start/end time, or had some other issue with the trip. Let’s remove them.

```
DELETE FROM bike_data
WHERE started_at = ended_at
```

Looking at the lowest end of the data, we see a lot of trips that lasted only 1 second.

```
SELECT COUNT(*)
FROM bike_data
WHERE ((julianday(ended_at) - julianday(started_at)) * 25 * 60 * 60) < 2
```
-> 1002

```
SELECT COUNT(*)
FROM bike_data
WHERE ((julianday(ended_at) - julianday(started_at)) * 25 * 60 * 60) < 10
```
-> 30221

With over 30,000 trips lasting less than 10 seconds, I’m hesitant to delete them just for being a short time. I can’t know exactly what caused so many people to rent and then immediately dock their bike, but it could say something about casual members who changed their mind, so at least for now I’ll leave these short trips.


### Analysis


Across all seasons, casual riders generally take longer rides than members:

```
SELECT member_casual, 
ROUND(AVG((julianday(ended_at) - julianday(started_at)) * 24 * 60), 2 ) AS avg_ride_length
FROM bike_data
GROUP BY 1
```

member_casual | avg_ride_length
--- | ---
casual | 21.19
member | 12.2


Across seasons, the number of casual riders and their average ride length varies greatly, peaking in spring and dropping in winter. Members maintain similar ride lengths across seasons and their number of riders fluctuates less greatly, though they also drop in ride count during winter.


```
WITH ride_length AS (
SELECT *,
ROUND((julianday(ended_at) - julianday(started_at)) * 24 * 60, 2 ) AS ride_length
FROM bike_data
),

season AS (
SELECT *,
CASE
WHEN  strftime("%m", started_at) BETWEEN "03" AND "05"
THEN "spring"
WHEN strftime("%m", started_at) BETWEEN "06" AND "08"
THEN "summer"
WHEN strftime("%m", started_at) BETWEEN "09" AND "11"
THEN "fall"
ELSE "winter" 
END AS season
FROM ride_length
ORDER BY random()
)

SELECT member_casual, season, AVG(ride_length), COUNT(*)
FROM season
GROUP BY 1, 2
```

member_casual | season | AVG(ride_length) | COUNT(*)
--- | --- | --- | ---
casual | fall | 18.6865479583801 | 605181
casual | spring | 22.6565723874716 | 488655
casual | summer | 22.6656805099861 | 1131328
casual | winter | 14.3127921792963 | 127610
member | fall | 11.8305077878633 | 990965
member | spring | 11.8489144315961 | 830026
member | summer | 13.3845952269197 | 1244228
member | winter | 10.3222802503794 | 434540



ride_distance_km AS (
SELECT *,
ROUND(
6371.0 * 2 * ASIN(
        SQRT(
            POWER(SIN(RADIANS(end_lat - start_lat) / 2), 2) +
            COS(RADIANS(start_lat)) * COS(RADIANS(end_lat)) * POWER(SIN(RADIANS(end_lng - start_lng) / 2), 2)
        )
    ), 3
)	AS distance_km
FROM season
)

This query implements the Haversine formula (https://www.movable-type.co.uk/scripts/latlong.html) to calculate distance between two lat/long points. 

There are limitations to applying this here. It is likely that most, if not all bike trips did not take a straight path from start location to end location. Many trips have the same start and end station, suggesting some people take a circular path and return to their start point. The distance we are calculating here is only the distance between the start and end points, not the ride distance. However, I think with the amount of data, this information is still useful, as we can distinguish between effective distance traveled between rider types.

 CASE CAST (strftime('%w', started_at) AS INTEGER)
  WHEN 0 THEN 'Sunday'
  WHEN 1 THEN 'Monday'
  WHEN 2 THEN 'Tuesday'
  WHEN 3 THEN 'Wednesday'
  WHEN 4 THEN 'Thursday'
  WHEN 5 THEN 'Friday'
  ELSE 'Saturday' END AS day_of_week,

This will label the day of the week of each trip.
For easier querying I will combine some of these into a larger query.

WITH analysis AS (
SELECT *,

--determine the season of the ride using the month from start time
CASE 
WHEN  strftime("%m", started_at) BETWEEN "03" AND "05"
THEN 'spring'
WHEN strftime("%m", started_at) BETWEEN "06" AND "08"
THEN 'summer'
WHEN strftime("%m", started_at) BETWEEN "09" AND "11"
THEN 'fall'
ELSE 'winter'
END AS season,

--determine the day of the week of the ride with strftime()
 CASE CAST (strftime('%w', started_at) AS INTEGER)
  WHEN 0 THEN 'Sunday'
  WHEN 1 THEN 'Monday'
  WHEN 2 THEN 'Tuesday'
  WHEN 3 THEN 'Wednesday'
  WHEN 4 THEN 'Thursday'
  WHEN 5 THEN 'Friday'
  ELSE 'Saturday' END AS day_of_week,
  
  --use the haversine formula to determine the net distance of the ride
  ROUND(
6371.0 * 2 * ASIN(
        SQRT(
            POWER(SIN(RADIANS(end_lat - start_lat) / 2), 2) +
            COS(RADIANS(start_lat)) * COS(RADIANS(end_lat)) * POWER(SIN(RADIANS(end_lng - start_lng) / 2), 2)
        )
    ), 3
)	AS distance_km,

--determine the total length of the ride
ROUND((julianday(ended_at) - julianday(started_at)) * 24 * 60, 2 ) AS length_min

FROM bike_data
)


SELECT ride_id, distance_km
FROM analysis
ORDER BY 2 DESC
LIMIT 100;



42AF82C53D831251
9814.069
E9495F1DC3475D41
9813.378
6AFE1471227BD76F
9813.072
75DE33501313D0CE
9812.918
7F49424E860E7094
9812.916
BB8AA29838266294
9812.174
3B47B333C0D186F0
9811.811
0A6988FE859F4D54
9811.511
353B37694B30396F
42.272
5CE2D7C544D25B78
37.679
D2A9A1120A165B1F
36.943
868D605CA8265D9C
36.512


Checking the maximum distance trips for all members, we get some huge outliers, which I will delete.






number of trips by season, member
number of trips by day of week, member
length of trip by season, member
length of trip by day of week, member



Popular stations —
