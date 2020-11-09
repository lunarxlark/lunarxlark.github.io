+++
title = "MySQLから各種一覧取得方法"
date = 2018-05-01
tags = ["MySQL"]
draft = false
+++


```sql
-- DB一覧
SHOW databases;
```

```sql
-- user一覧
SELECT host, user FROM user;
```

```sql
-- ### MySQL内の権限一覧 ###
-- グローバルレベル権限のリスト
SELECT * FROM information_schema.user_privileges;

-- データベースレベル権限のリスト
SELECT * FROM information_schema.schema_privileges;

-- テーブルレベル権限のリスト
SELECT * FROM information_schema.table_privileges;

-- カラムレベル権限のリスト
SELECT * FROM information_schema.column_privileges;
```

```sql
-- ユーザ別の権限一覧
SHOW GRANTS FOR '<user>'@'<db>';
```
