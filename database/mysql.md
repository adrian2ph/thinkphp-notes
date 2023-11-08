---
description: mysql5.7
---

# mysql

##

## timezone&#x20;

* Server Time Zone

You can set the default time zone for the MySQL server using the `default_time_zone` system variable.

* Session Time Zone:

Each client session can have its own time zone setting using the `SET time_zone` statement.

```sql
SELECT @@GLOBAL.time_zone, @@SESSION.time_zone;
SET time_zone = '+08:00';
```



* different timestamps due to different time\_zone

```sql
SET time_zone = 'UTC';
select UNIX_TIMESTAMP(DATE ( '2023-09-08' )) as utc;
SET time_zone = '+08:00';
select UNIX_TIMESTAMP(DATE ( '2023-09-08' )) as shanghai;
-- result: utc 1694131200 shanghai 1694102400
```



## mysql datatype



### datetime



* unix timestamp and datetime

1.  unix timestamp to date

    FROM\_UNIXTIME

```sql
SET time_zone = '+08:00';
SELECT
	FROM_UNIXTIME(addTime)
FROM
	think_tourists 
limit 5
```

1. date to unix timestamp
   1. ```
      UNIX_TIMESTAMP
      ```

```sql
SET time_zone = '+08:00';
SELECT
	count( free_id ) 
FROM
	think_tourists 
WHERE
		addtime >= UNIX_TIMESTAMP( DATE ( '2023-11-05' ) ) 
		AND addtime < UNIX_TIMESTAMP( DATE ( '2023-11-05' ) )+ 86400;
```



### json



* longtext to json

```sql
SELECT JSON_EXTRACT(longtext_column, '$') AS json_column FROM your_table;
```



* children nodes

JSON\_ARRAYAGG

```sql

SELECT
    cc.platform_type as id,
    p.`name` AS `name`,
		JSON_ARRAYAGG(JSON_OBJECT('name', cc.channel_name, 'id', cc.client_channel)) as channel
FROM
    think_client_channel AS cc
LEFT JOIN think_platform AS p ON cc.platform_type = p.type
WHERE
    cc.type = 2
GROUP BY
    platform_type
ORDER BY platform_type,client_channel


```
