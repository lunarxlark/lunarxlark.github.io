---
title : Javaのチューニングオプション
tags : ["Java"]
date : 2018-12-02
published : true
---

### Options

- `-D`はシステムのオプション
- `-XX:`はJava Hotspot VMのオプション [# Optinos_list](https://www.oracle.com/technetwork/java/javase/tech/vmoptions-jsp-140102.html)
- `-XX:`オプションに対して、`+`で有効、`-`で無効を表す。

<!--more-->

```shell
### memory ###
# Permanent領域の最大サイズを指定
-XX:MaxPermSize=9g

# Permanent領域の初期サイズを指定()
-XX:PermSize=9g
```

```shell
### Garbage Collect ###
# GCの出力情報をファイルに保存
-Xloggc:/path/to/gc.log

# GCの詳細情報を出力
-XX:+PrintGCDetails

# GC開始時刻追記
-XX:+PrintGCDateStamps
```

```shell
### JIT Compiler ###
# JITコンパイラでコンパイルされた各メソッドの情報をログ・ファイルに記録
-XX:+PrintCompilation

# メソッドがコンパイルされるための条件となるメソッド呼び出し回数を設定
-XX:CompileThreshold=n

# 使用するコードキャッシュの総サイズを設定
-XX:ReservedCodeCacheSize=YYm

# 利用頻度の低いコード・ブロブのフラッシュをJVMに許可(Java7 Update4以降ではdefault:on)
-XX:+UseCodeCacheFlushing

## コンパイル閾値を下げたくない。でも、重要なメソッドを可能な限り早急にコンパイルしたい場合の対応
ウォームアップ処理 … プロセス起動後にテスト・トラフィックを送信することでコードを十分な回数実行して強制的にコンパイルさせる
```

### ref

- :star: [Akira's Tech Notes [tips][Java]CodeCache領域使用状況の確認方法](http://luozengbin.github.io/blog/2015-09-01-%5Btips%5D%5Bjava%5Dcodecache%E9%A0%98%E5%9F%9F%E4%BD%BF%E7%94%A8%E7%8A%B6%E6%B3%81%E3%81%AE%E7%A2%BA%E8%AA%8D%E6%96%B9%E6%B3%95.html)
- :star: [Java HotSpot VM コードキャッシュについて](https://www.oracle.com/webfolder/technetwork/jp/javamagazine/Java-JA13-Architect-evans.pdf)
