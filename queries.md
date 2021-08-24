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


## finding popular parsers in last 4 months 


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
