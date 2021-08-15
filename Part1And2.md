## Part 1 - Querying Data with BigQuery

- What's the size of this dataset? (i.e., how many trips)

The size of the dataset is 983648 trips.

`
SELECT count(DISTINCT trip_id) FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
`

- What is the earliest start date and time and latest end date and time for a trip?

The earliest start date is 2013-08-29 09:08:00 UTC and latest end date is 2016-08-31 23:48:00 UTC.

`
SELECT min(start_date) as earliest_date, max(end_date) as latest_date FROM `bigquery-public-data.san_francisco.bikeshare_trips`; 
`

- How many bikes are there?

There are 700 unique bikes.

`
SELECT count(distinct bike_number) as total_bikes FROM `bigquery-public-data.san_francisco.bikeshare_trips`; 
`

#### Questions of your own

- Subscriber Type is Subscriber = annual or 30-day member; Customer = 24-hour or 3-day member. How many trips were taken by each subscriber type. What is the average trip duration by each subscriber type. 
  
  * Answer:  
  Subscriber type of Customer took a total of 136809 trips with an average duration of 61.97 minutes. Subscriber Type of Subscriber took a total of 846839 trips with an average duration of 9.71 minutes. 
  
  * SQL query: 
`
 SELECT subscriber_type,count(trip_id) as total_trips, avg(duration_sec)/60 as avg_duration_mins FROM `bigquery-public-data.san_francisco.bikeshare_trips` GROUP BY subscriber_type
`

- Question 2: Which Station has the maximum number of docks?
  * Answer: 
  Cyril Magnin St at Ellis St station near landmark has 35 docks.
  
  * SQL query:
`
SELECT name, dockcount, LANDMARK FROM bigquery-public-data.san_francisco.bikeshare_stations
ORDER BY dockcount DESC
limit 1
`

- Question 3: Which station has highest number available bikes on average.
  * Answer: 
  5th St at Folsom St has 16.64 available bikes on average.

  * SQL query:
`
SELECT name FROM `bigquery-public-data.san_francisco.bikeshare_stations` WHERE station_id =
(SELECT station_id
FROM `bigquery-public-data.san_francisco.bikeshare_status` 
group by station_id ORDER BY avg(bikes_available) desc LIMIT 1)
`

```sql
Create view uplifted-module-312802.test.bikes_station_region as (select distinct region_id, station_id
from `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`)
```

**** Top 5 station pairs  

```sql
Create view uplifted-module-312802.test.bikes_station_region as (select distinct region_id, station_id
from `bigquery-public-data.san_francisco_bikeshare.bikeshare_station_info`)
```

```sql
SELECT start_station_name, end_station_name, count(*) as trip_freq FROM `bigquery-public-data.san_francisco.bikeshare_trips` GROUP BY start_station_name, end_station_name ORDER BY trip_freq DESC LIMIT 5
```

Answer:
1	
Harry Bridges Plaza (Ferry Building)
Embarcadero at Sansome
9150
2	
San Francisco Caltrain 2 (330 Townsend)
Townsend at 7th
8508
3	
2nd at Townsend
Harry Bridges Plaza (Ferry Building)
7620
4	
Harry Bridges Plaza (Ferry Building)
2nd at Townsend
6888
5	
Embarcadero at Sansome
Steuart at Market
6874


***** Top 3 regions

3,12,5 are the top regions. 

```sql
With example as 
(
SELECT count(trip_id) as c, start_station_id
FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
GROUP BY start_station_id 
)
Select count(c), region_id from example 
join uplifted-module-312802.test.bikes_station_region as b on 
example.start_station_id = b.station_id
group by(region_id)
```


## Part 2 - Querying data from the BigQuery CLI 

- Use BQ from the Linux command line:

  * General query structure

    ```
    bq query --use_legacy_sql=false '
        SELECT count(*)
        FROM
           `bigquery-public-data.san_francisco.bikeshare_trips`'
    ```

**** Part 2

* What's the size of this dataset? (i.e., how many trips)

    ```
    bq query --use_legacy_sql=false 'SELECT count(DISTINCT trip_id) FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
    ```
    
Answer:
+-----------+
| all_trips |
+-----------+
|    983648 |
+-----------+

* What is the earliest start time and latest end time for a trip?

    ```
    bq query --use_legacy_sql=false 'SELECT min(start_date) as earliest_date, max(end_date) as latest_date FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
    ```
Answer:
+---------------------+---------------------+
|    earliest_date    |     latest_date     |
+---------------------+---------------------+
| 2013-08-29 09:08:00 | 2016-08-31 23:48:00 |
+---------------------+---------------------+

* How many bikes are there?

    ```
    bq query --use_legacy_sql=false 'SELECT count(distinct bike_number) as total_bikes FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
    ```
Answer:
+-------------+
| total_bikes |
+-------------+
|         700 |
+-------------+

 * How many trips are in the morning vs in the afternoon?
 
 Morning is defined between 6 am to 12 pm and evening is defined between 5 pm and 9 pm. There are 6 hours in the morning window and 4 hours in the evening window.
 
     ```
    bq query --use_legacy_sql=false 'SELECT count(EXTRACT(hour FROM start_date)) as morning_trips
FROM `bigquery-public-data.san_francisco.bikeshare_trips` Where EXTRACT(hour FROM start_date) Between 6 and 12;'

    bq query --use_legacy_sql=false 'SELECT count(EXTRACT(hour FROM start_date)) as evening_trips
FROM `bigquery-public-data.san_francisco.bikeshare_trips` Where EXTRACT(hour FROM start_date) Between 17 and 21'
    ```
Answer:
+---------------+
| morning_trips |
+---------------+
|        446771 |
+---------------+

+---------------+
| evening_trips |
+---------------+
|        289947 |
+---------------+

### Project Questions
Identify the main questions you'll need to answer to make recommendations (list
below, add as many questions as you need).

- Question 1: What are the rush (peak) hours in the data.

- Question 2: What are the most popular stations on weekends?

- Question 3: Is there a particular hour when the bikes are idle

- Question 4: Is there a preference for the time slots amoung subscribers?

- Question 5: Is trip Duration different among the two subscribers?

- Question 6: What is the usage percentage by hour in each region? Usage percentage can be calculated as number of bikes/number of slots

- Question 7: Do subscribers do less or more commuter trips as compared to customers?

- Question 8: Do subscribers ride more on weekdays as compared to weekends?

- Question 9: Do customers have longer durations trips as compared to subscribers on average?

### Answers

- Question 1: What are the rush hours in the data.
  
  Answer: The rush hours are 7 am to 9 am in the morning and 5pm to 7 pm in the evening. 
	trips	commute_hour
0	132464	8
1	126302	17
2	96118	9
3	88755	16
4	84569	18
5	67531	7

* SQL query:
```
SELECT count(trip_id) as trips, EXTRACT(hour FROM start_date) as commute_hour
FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
group by commute_hour
order by trips DESC LIMIT 6
```

- Question 2: Is there a particular hour and location, when and where, the bikes are idle?

Answer:

On average 20 bikes remain idle between 6pm to 8pm on 5th St at Folsom St in San Francisco. 
On average 16 bikes remain idle from 6 pm onwards on  Harry Bridges Plaza (Ferry Building) in San Francisco.

trip_hour	bike_avail	station_id	name	landmark	
1	19 -> 20.616541353383457
90
5th St at Folsom St
San Francisco
2	
0
20.103151862464184
90
5th St at Folsom St
San Francisco
3	
18
20.071246819338416
90
5th St at Folsom St
San Francisco
4	
2
20.04385964912281
90
5th St at Folsom St
San Francisco
5	
5
20.0
90
5th St at Folsom St
San Francisco
6	
1

```
SELECT EXTRACT(hour FROM time) as trip_hour, avg(bikes_available) as bike_avail, 
a.station_id, b.name, landmark
FROM `bigquery-public-data.san_francisco.bikeshare_status` a JOIN 
`bigquery-public-data.san_francisco.bikeshare_stations` b ON 
a.station_id = b.station_id
group by trip_hour, station_id, name, landmark
order by 2 DESC

```

- Question 3: Do subscribers ride more on weekdays as compared to weekends?
  
  Answer: Yes, Subscribers prefer weekdays over weekends. 
  	subscriber_type	day_of_week	trips
0	Subscriber	Tue	169668
1	Subscriber	Wed	165530
2	Subscriber	Thu	160296
3	Subscriber	Mon	154795
4	Subscriber	Fri	140048
5	Subscriber	Sat	31035
6	Customer	Sat	29244
7	Customer	Sun	25908

```
CREATE TEMPORARY FUNCTION dayOfWeek(x TIMESTAMP) AS(  ['Sun','Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']      [ORDINAL(EXTRACT(DAYOFWEEK from x))]);

SELECT subscriber_type, dayOfWeek(start_date) as day_of_week , COUNT(trip_id) as trips 
FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
group by day_of_week, subscriber_type
Order by trips DESC
```

- Question 4: Do customers have longer durations trips as compared to subscribers on average?

Answer: Yes. Customers have longer duration trips. Customers have 61.9 minutes and Subscribers have 9.7 minutes on average

1	
Customer
136809
61.97975267221707
2	
Subscriber
846839
9.712737328662037

```
SELECT count(trip_id) as trips, avg(duration_sec) / 60 AS minutes,  subscriber_type 
FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
group by subscriber_type
```