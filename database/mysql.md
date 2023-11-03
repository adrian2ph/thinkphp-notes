---
description: mysql5.7
---

# mysql

## datetime

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

