+++
title = "SQL*Plusでviを使う"
date = 2018-12-01
tags = ["SQL*PLUS", "vim"]
draft = false
+++

```sql
SQL> DEFINE _EDITOR = vi
SQL> edit
SQL> /   -- 実行
```

glogin.sqlに設定することで`DEFINE _EDITOR = vi`を省略できる
