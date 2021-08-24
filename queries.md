# PIPY Queries

## no. of downloads of my package('spotify-web-controller') in last 4 months 

```sql
SELECT COUNT(*) AS num_downloads
FROM `bigquery-public-data.pypi.file_downloads`
WHERE file.project = 'spotify-web-controller'
  AND DATE(timestamp)
    BETWEEN DATE_SUB(CURRENT_DATE(), INTERVAL 120 DAY)
    AND CURRENT_DATE()
    
```


![image](https://user-images.githubusercontent.com/37925362/130643342-840c5046-8ac5-4787-a792-d49a362f47e8.png)


## finding popular parsers in last 4 months  over the past 4 months


```sql
SELECT
       file_downloads.file.project  AS Project_name,
       COUNT(*) AS file_downloads_count
FROM `bigquery-public-data.pypi.file_downloads`
  AS file_downloads
WHERE (file_downloads.file.project LIKE '%parser%') AND (file_downloads.timestamp >= timestamp_add(current_timestamp(), INTERVAL -(4*30) DAY)) 
GROUP BY Project_name
ORDER BY file_downloads_count DESC
    
```

![image](https://user-images.githubusercontent.com/37925362/130644751-6397b80b-214c-48df-84ca-39ccfaabb648.png)

## Top countries with highest package downloads over the past 4 months
```sql
SELECT
       country_code,
       COUNT(*) AS file_downloads_count
FROM `bigquery-public-data.pypi.file_downloads`
  AS file_downloads
WHERE (file_downloads.timestamp >= timestamp_add(current_timestamp(), INTERVAL -(4*30) DAY)) 
GROUP BY country_code
ORDER BY file_downloads_count DESC
limit 10
    
```

![image](https://user-images.githubusercontent.com/37925362/130646864-2c443b57-118c-4b35-917c-1f4ec5e931b8.png)



## Average downloads per package by country over the past 4 months
```sql
with pacakge_downloads as 
(
SELECT
       file_downloads.file.project  AS Project_name,
       COUNT(*) AS file_downloads_count
FROM `bigquery-public-data.pypi.file_downloads`
  AS file_downloads
WHERE (file_downloads.file.project LIKE '%parser%') AND (file_downloads.timestamp >= timestamp_add(current_timestamp(), INTERVAL -(4*30) DAY)) 
GROUP BY Project_name
ORDER BY file_downloads_count DESC
)


SELECT country_code,
       AVG(pacakge_downloads.file_downloads_count) AS file_downloads_avg
FROM `bigquery-public-data.pypi.file_downloads`
  AS file_downloads
join pacakge_downloads
on pacakge_downloads.Project_name = file_downloads.file.project

WHERE  (file_downloads.timestamp >= timestamp_add(current_timestamp(), INTERVAL -(4*30) DAY)) 
GROUP BY country_code
ORDER BY file_downloads_avg DESC

```


![image](https://user-images.githubusercontent.com/37925362/130653058-6841c864-4672-425e-92c6-e7b039a5f0d0.png)




## Top packages of top 5 countries with downloads greater than avg downloads of that country over the past 4 months
```sql
with pacakge_downloads as 
(
SELECT
       file_downloads.file.project  AS Project_name,
       COUNT(*) AS file_downloads_count
FROM `bigquery-public-data.pypi.file_downloads`
  AS file_downloads
WHERE (file_downloads.file.project LIKE '%parser%') AND (file_downloads.timestamp >= timestamp_add(current_timestamp(), INTERVAL -(4*30) DAY)) 
GROUP BY Project_name
ORDER BY file_downloads_count DESC
),

count_avg as
(
SELECT country_code,
       AVG(pacakge_downloads.file_downloads_count) AS file_downloads_avg
FROM `bigquery-public-data.pypi.file_downloads`
  AS file_downloads
join pacakge_downloads
on pacakge_downloads.Project_name = file_downloads.file.project

WHERE  (file_downloads.timestamp >= timestamp_add(current_timestamp(), INTERVAL -(4*30) DAY)) 
GROUP BY country_code
ORDER BY file_downloads_avg DESC
),

top5 as (

SELECT
       country_code,
       COUNT(*) AS file_downloads_count
FROM `bigquery-public-data.pypi.file_downloads`
  AS file_downloads
WHERE (file_downloads.timestamp >= timestamp_add(current_timestamp(), INTERVAL -(4*30) DAY)) 
GROUP BY country_code
ORDER BY file_downloads_count DESC
limit 5
),
count_proj as (
SELECT top5.country_code,
       file_downloads.file.project  AS Project_name,
       COUNT(*) AS file_downloads_count
FROM `bigquery-public-data.pypi.file_downloads`
  AS file_downloads
join top5
on top5.country_code = file_downloads.country_code 

WHERE  (file_downloads.timestamp >= timestamp_add(current_timestamp(), INTERVAL -(4*30) DAY)) 
GROUP BY country_code,Project_name
ORDER BY top5.country_code
)

SELECT *
FROM count_proj
join count_avg
on count_avg.country_code = count_proj.country_code 

WHERE  count_proj.file_downloads_count >=count_avg.file_downloads_avg
ORDER BY count_proj.file_downloads_count DESC
```

![image](https://user-images.githubusercontent.com/37925362/130656692-c4247e0e-4cff-44ed-9107-b70e74e819e3.png)
