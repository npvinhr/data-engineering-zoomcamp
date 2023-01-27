## Week 1 Homework

In this homework we'll prepare the environment and practice with Docker and SQL

https://github.com/DataTalksClub/data-engineering-zoomcamp/blob/main/cohorts/2023/week_1_docker_sql/homework.md




## Question 1. Knowing docker tags

Run the command to get information on Docker build:

```docker build --help```




## Question 2. Understanding docker first run

Run docker with the ```python:3.9``` image in an interactive mode and the entrypoint of bash.
Now check the python modules that are installed ( use ```pip list```).

```docker run -it python:3.9 bash```
```pip list```




# Prepare Postgres

Run Postgres and pgAdmin with Docker-compose (the setting is in ```docker-compose.yaml``` file):

```docker-compose up -d```


Run python script ```ingest_data.py``` to ingest the data in chunk:

```
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-01.csv.gz"

python ingest_data.py \
  --user=root \
  --password=root \
  --host=localhost \
  --port=5432 \
  --db=ny_taxi \
  --table_name=green_taxi_trips \
  --url="${URL}"
 ```


Import the 'zones' table via Jupyter notebook using ```pandas``` and ```sqlalchemy```

```Postgres_ny_taxi - connect.ipynb```


Connect to pgAdmin via browser to query the database for Question #3-#6:

```localhost:8080```




## Question 3. Count records 

How many taxi trips were totally made on January 15?

```
SELECT 
	COUNT(*)
FROM 
	green_taxi_trips
WHERE
	CAST(lpep_pickup_datetime AS DATE) = '2019-01-15'
	AND
	CAST(lpep_dropoff_datetime AS DATE) = '2019-01-15';
```




## Question 4. Largest trip for each day

Which was the day with the largest trip distance

```
SELECT 
	CAST(lpep_pickup_datetime AS DATE) AS pickup_date,
	trip_distance
FROM 
	green_taxi_trips
ORDER BY
	trip_distance DESC
LIMIT 10;
```




## Question 5. The number of passengers

In 2019-01-01 how many trips had 2 and 3 passengers?

```
SELECT 
	COUNT(*)
FROM 
	green_taxi_trips
WHERE
	CAST(lpep_pickup_datetime AS DATE) = '2019-01-01'
	AND
	passenger_count = 2;
```

```
SELECT 
	COUNT(*)
FROM 
	green_taxi_trips
WHERE
	CAST(lpep_pickup_datetime AS DATE) = '2019-01-01'
	AND
	passenger_count = 3;
```




## Question 6. Largest tip

For the passengers picked up in the Astoria Zone which was the drop off zone that had the largest tip?

```
SELECT 
	zpu."Zone" AS "pickup_zone",
	zdo."Zone" AS "dropoff_zone",
	tip_amount
FROM
	green_taxi_trips t
	JOIN zones zpu
		ON t."PULocationID" = zpu."LocationID"
	JOIN zones zdo
		ON t."DOLocationID" = zdo."LocationID"
WHERE
	zpu."Zone" = 'Astoria'
ORDER BY
	tip_amount DESC
LIMIT 10;
```

