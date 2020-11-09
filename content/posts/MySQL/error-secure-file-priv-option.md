+++
title = "dockerでmysqlイメージ起動時に起きたエラー [ERROR] secure_file_priv"
date = 2018-11-29
tags = ["MySQL"]
draft = false
+++


 [MySQL ERROR 1290 (HY000) --secure-file-priv option - Stack Overflow](https://stackoverflow.com/questions/34102562/mysql-error-1290-hy000-secure-file-priv-option/34102667)

```sql
  mysql> load data infile 'account.csv' into table authapi.account   fields     terminated by ','     enclosed by '"';  
  ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
  
  mysql> SELECT @@GLOBAL.secure_file_priv;
  +---------------------------+
  | @@GLOBAL.secure_file_priv |
  +---------------------------+
  | /var/lib/mysql-files/     |
  +---------------------------+
  ```
