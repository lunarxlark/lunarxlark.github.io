+++
title = "BigQueryでPARTITIONTIMEを比較対象にする場合の記述"
date = 2019-01-10
tags = ["BigQuery"]
draft = false
+++

比較対象はtimestamp型に変更してから比較

```sql
SELECT
  *
FROM
  [zzzzzzzzzz.xxxxxxxxxxxxx]
WHERE
  _PARTITIONTIME between timestamp('2018-12-04') and timestamp('2018-12-11')
ORDER BY log_time DESC
LIMIT 100
```
