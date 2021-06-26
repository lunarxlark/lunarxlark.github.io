+++
title = "MySQLへcsvをloadする方法"
date = 2018-06-15
tags = ["MySQL"]
draft = false
+++


```sql
  load data infile 'account.csv' into table authapi.account
    fields
      terminated by ','
      enclosed by '\\"'
      escaped by '\\'
  lines
    terminated by '\\\\n'
```
