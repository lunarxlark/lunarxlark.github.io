+++
title = "LSP(Language Server Protocol)のドキュメントを読む"
date = 2019-02-23T10:07:17+09:00
tags = ["lsp", "jsonRPC"]
draft = false
+++


golangコミュニティが[gopls](https://godoc.org/golang.org/x/tools/cmd/gopls)を開発し始めたり、Microsoftが提唱したこともあって`Language Server Protocol (以後,LSP)`が注目され始めた。
特にVimやgolang界隈ではmattnさんのブログ記事[gocode やめます(そして Language Server へ)](https://mattn.kaoriya.net/software/lang/go/20181217000056.htm)で一層注目を集めたと思う。是非読んで欲しい。
gocode contribuerへの思いとLSP発展への期待が書かれている。


そんな熱いLSPを理解することでVim発展に拍車がかかることへの期待とgoplsを理解したいという興味から、まずはLSPドキュメントを訳してみた。
実際に、vim-lsp+goplsを入れて、log出力と照らし合わせながらLSPがどういう風に動くものなのか、理解が深まったと思う。以下、ドキュメントの訳。

<!--more-->


**意訳しかないので、正しく理解するためには公式ドキュメントを参照してください。**

[LSP Overview](https://microsoft.github.io/language-server-protocol/overview)


---
### LSPって何なん?
> 自動補完や定義ジャンプ、ホバーなどを実現するために、プログラミング言語毎に偉大な努力が費やされてきました。
> 歴史的に、これらの作業は開発ツール毎に繰り返され、同じ機能を異なるAPIで提供されていました。

`Language Server`はプログラミング言語特有の便利機能や開発ツールを内部プロセス間プロコルで提供します。


> このアイディアの背景には、サーバと開発ツール間のコミュニケーションの標準化があります。
> こうすることで、１つの`Language Server`は複数の開発ツールで再利用され、続いて最小限の労力で複数言語をサポート出来ます。

LSPはプログラミング言語提供者とツール開発者両方にとって希望の光なのです!


---
### どうやって動くん?

language serverは分割されたプロセスで動き(?)、開発者ツールとはlanguage protocol越しにJSON_RPCを使ってやりとりします。
下記の図は、編集時のセッション間でlanguage serverがどうやって開発ツールとlanguage serverがっやりとりするかを示したサンプルです。

![sample:editing session](https://microsoft.github.io/language-server-protocol/img/language-server-sequence.png)

- 開発ツールでファイル開いた時(ドキュメントとして参照した時)

ツールはlanguage serverにドキュメントが開かれたことを知らせます(`textDocument/didOpen`)。
この時から、本物のドキュメントはもうファイルシステム上のものではなく、ツールによってメモリに保持されています。
それらのコンテンツはツールとlanguage server間で同期されなければいけません。

- 編集時

ツールはlanguage serverにドキュメントが変更されたことを知らせて(`textDocument/didChange`)、language serverによってドキュメントの記述が更新されます。
この時、language serverは更新されたドキュメントの情報を解析して、ツールにエラーや警告を知らせます(`textDocument/publishDiagnostics`)。

- ドキュメントを開くシンボル上で定義ジャンプを実行した時

ツールは`textDocument/definition`を2つのパラメータ(1:ドキュメントURI、2:定義ジャンプをlanguage serverに要求した時のテキスト位置)を送ります。
language serverはドキュメントURIとドキュメント内のシンボル定義位置を返します。


(vim-lsp+goplsのログから抜粋)

```json
# tool -> language server
["--->", 4, "gopls", {"method": "textDocument/definition", "on_notification": "---funcref---", "params": {"textDocument": {"uri": "file:///home/vagrant/dev/src/github.com/lunarxlark/echo-learning/main.go"}, "position": {"character": 5, "line": 5}}}]

# language server -> tool
["<---", 4, "gopls", {"response": {"id": 2, "jsonrpc": "2.0", "result": [{"uri": "file:///usr/local/go/src/fmt/print.go", "range": {"end": {"character": 12, "line": 262}, "start": {"character": 5, "line": 262}}}]}, "request": {"method": "textDocument/definition", "jsonrpc": "2.0", "id": 2, "params": {"textDocument": {"uri": "file:///home/vagrant/dev/src/github.com/lunarxlark/echo-learning/main.go"}, "position": {"character": 5, "line": 5}}}}]
```

ref: [JSON_RPC 2.0 Specification](https://www.jsonrpc.org/specification)

- ドキュメント(ファイル)を閉じる時

`textDocument/didClose`の通知がツールからlanguage serverへ送られ、この時にドキュメントはメモリからいなくなります。
編集していた(つもりの!)コンテンツはこの時にファイルシステム上で更新されます。

---

サンプルからわかったように、ツールとlanguage server間のやりとり内容はとてもシンプルで、どういうデータをやり取りするかというのは言語に依りません。

---


### どんな機能あるん?

全てのプログラミング言語がこのprotocolで全機能をサポート出来るわけではありません。
そのため、LSPでは`機能のグループ`を提供します。機能のグループは、各プログラミング言語の特有の機能の集まりです。
開発ツールやlanguage serverはサポートする機能を知らせてくれます。例えば、`textDocument/definition`リクエストを扱えるとサーバが約束したら、`workspace/symbol`リクエストは使えないかもしれないということです。
同様に、開発ツールが`about to save`通知をドキュメント保存時に提供してくれる機能を約束してくれることで、language serverは保存時に編集されたドキュメントをフォーマットして保存することが出来ます。


#### 注意

**特定ツールへlanguage server組み込む方法はLSPでは決めていません。各ツールでの実装として残っています。**



(次回、LSP Specificationへ続く)
