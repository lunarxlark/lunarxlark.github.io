+++
title = "sudoの設定ファイル:sudoersのまとめ"
date = 2018-11-14
tags = ["Linux", "sudoer", "visudo"]
draft = false
+++


- sudoers ... sudoに関する設定ファイル
- visudo ... sudoersを編集するためのコマンド

<!--more-->

下記のように、各ユーザの権限内容を記載したsudoersはrootでも`+r`しかない。  
そのため、`visudo`コマンドでsudoersを編集する。  

```bash
[vagrant@lunarxlark ~]# ll /etc/sudoers
-r--r----- 1 root root 4200  1月 15 17:42 2019 /etc/sudoers
```


`誰[%が付くとグループ]`が`どこ[ホスト名等]`から`=(誰[カンマでグループも指定可能])`になって`実行出来るコマンド`を表す。

```shell
### visudo時 ###

# root(wheelグループ)は 全接続先から=全ユーザで 全コマンドを実行出来る
root      ALL=(ALL)    ALL
%wheel    ALL=(ALL)    ALL

# hogeは 全接続先からfugaで 全コマンドを実行出来る
hoge     ALL=(fuga)    ALL

# hogeは sudo -u fugaをパスワード無しで実行出来る
hoge     ALL=(fuga)    NOPASSWD: ALL

# hogeは sudo -u fugaをパスワード有りで実行出来る
hoge     ALL=(fuga)    PASSWD: ALL

# apache関連のコマンドのみ許可
hoge     ALL=(root)    PASSWD: /etc/init.d/httpd

## コマンドエイリアス
Cmnd_Alias APACHE /etc/init.d/http
hoge     ALL=(root)    NOPASSWD: APACHE
```

## Ref

- [sudoの設定ふぁいるsudoersについて ユニファ開発者ブログ](https://tech.unifa-e.com/entry/2018/05/18/112810)
- [sudoユーザを追加する方法](https://webkaru.net/linux/sudo-user-add/)
