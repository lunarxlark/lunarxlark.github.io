+++
title = "実行している現在のMakefileのディレクトリ取得方法"
date = 2018-07-12
tags = ["Makefile"]
draft = false
+++

```Makefile
MAKEFILE_DIR := $(dir $(lastword $(MAKEFILE_LIST)))
```
