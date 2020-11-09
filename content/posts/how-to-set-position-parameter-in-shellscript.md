+++
title = "setコマンドでの位置パラメータへの代入方法"
date = 2018-08-03
tags = ["ShellScript"]
draft = false
+++

--オプションを使う事で、文字列を空白文字で分割して、位置パラメータにセットできる。

<!--more-->

```bash
set -- $line    # line <= "hoge fuga"
$chunk1=$1      # hoge
$chunk2=$2      # fuga
```
