---
title : 自分用日本語訳 Go Module
tags : ["go"]
published : 2020-05-14
---

(この記事は、執筆時点での[Go Modules](https://github.com/golang/go/wiki/Modules)をもとにした私のメモです。Go Moduleの理解のために、元記事と記載の順番が入れ替わっている項目もあります。)

<!--more-->

Go言語は, 1.11以降はモジュールのバージョン管理をサポートしています。初期のプロトタイプ `vgo` がアナウンスされたのは2018/2です。2018/5にバージョン管理されたモジュールが初めてgoのリポジトリで使用されだしました。

Go1.14で、モジュールサポートは本番利用に向けて考慮され, 全goユーザは他の依存管理システムからモジュールへ以降することが推奨されました。もし、goのツールチェインの問題で以降できない場合、その問題が起票されているか確認してください。
(もし、その起票が1.15のマイルストーンに載っていない場合、問題解決の優先順位を適切にするために、その問題であなたのモジュール移行がどうして妨害されているのかコメントしてください。 ) より詳細なフィードバックをするために [experience report](https://github.com/golang/go/wiki/ExperienceReports) も提供されています。


## Quick Start

### Example

詳細はこのページの以降に記載するとして、ここでは初めから作成する簡単なモジュールの例を取り上げる。

1. $GOPATHの外でディレクトリを作成して、適宜VCSも初期化してください。

```bash
$ mkdir -p /tmp/scratchpad/repo
$ cd /tmp/scratchpad/repo
$ git init -q
$ git remote add origin https://github.com/my/repo
```

2. moduleの初期化

```bash
$ go mod init github.com/my/repo
go: creating new go.mod: module github.com/my/repo
```

3. コードを書きます。

```go:hello.go
package main

import (
  "fmt"
  "rsc.io/quote"
)

func main() {
  fmt.Println(quote.Hello())
}
```

4. build and run.

```bash
$ go build -o hello
$ ./hello

Hello, world.
```

5. `go.mod` が更新され、依存関係にあるバージョンが明示的に記載される。 ここの `1.5.2` は[セマンティックバージョン体系](https://semver.org/)になっている。


```go:go.mod
module github.com/my/repo

require rsc.io/quote v1.5.2
```


### Daily workflow

注意して欲しいのが、上記の例では `go get` がなかった。
あなたの典型的な日々のworkflowは以下のようになるだろう。

- `*.go` に必要なimport行を追加する。
- 標準的なコマンド(`go build` or `go test`)がimportステートメントを満たすように必要な新しい依存関係を自動的に追加する。
- 必要であれば、依存関係にあるライブラリの特定バージョンを次のようなコマンドで選ぶことができるし、また `go.mod` を直接編集して指定できる。
  - `go get foo@v1.2.3`
  - `go get foo@master`
  - `go get foo@3702bed2`


その他の共通機能に関する短いツアーで以下を使う。

- `go list -m all` --- 直接/間接的な依存関係に対してビルドで使用されるライブラリの最終バージョンを表示する。[detail:Version selection](https://github.com/golang/go/wiki/Modules#version-selection)
- `go list -u -m all` --- 直接/間接的な依存関係に対して利用可能なパッチ/マイナーのアップグレードを表示する。 [detail:How to upgrade and downgrade dependencies](https://github.com/golang/go/wiki/Modules#how-to-upgrade-and-downgrade-dependencies)
- `go get -u ./...` or `go get -u=pathc ./...` (from module root directory) --- すべての直接/間接的な依存関係にあるライブラリを最新のバージョンにアップデートします。(pre-releaseは除く)
- `go build ./...` or `go test ./...` (from  module root directory) --- build or testをモジュール内の全パッケージで実施します。[detail:How to define a module](https://github.com/golang/go/wiki/Modules#how-to-define-a-module)
- `go mod tidy` --- もう使われていない依存関係のライブラリを`go.mod`から削除し、OS/arch/build tagのその他の組み合わせで必要な依存関係を追加します。[detail:How to prepare for a release](https://github.com/golang/go/wiki/Modules#how-to-prepare-for-a-release)
- `replace`ディレクティブ or `gohack` --- forkやローカルにコピーしたライブラリ、正確なバージョンのライブラリを使います。
- `go mod vendor` --- `vendor`ディレクトリを作成する任意の手順です。[detail:How do I use vendoring with modules is vendoring going away](https://github.com/golang/go/wiki/Modules#how-do-i-use-vendoring-with-modules-is-vendoring-going-away)


次のセクション`New Concepts`を読めば、ほとんどのプロジェクトでモジュールを始めるのに必要な情報が揃ったことになります。また上の[Table of Contents]を見直すことは詳細な内容に慣れるのにも役立ちます。


## New Concepts

これらのセクションは主な新しいコンセプトに対する高レベルの説明になります。詳細なコンセプトと根拠は[40分のビデオ](https://www.youtube.com/watch?v=F8nrpe0XWRg&list=PLq2Nv-Sh8EbbIjQgDzapOFeVfv5bGOoPE&index=3&t=0s)と[公式の提案](https://golang.org/design/24301-versioned-go)を見るか、より詳細な初期の[vgo blogシリーズ](https://research.swtch.com/vgo)を見てください。


### Modules

**module**は関連するGoのパッケージ(1つの集まりとしてバージョン付けされた)達の集合です。

moduleらは正確な依存関係を記録し、再現可能なビルドを作成します。

よくあるのが、バージョン管理リポジトリがリポジトリのルートにたった一つに定まったモジュールを含んでいることです。
([1つのリポジトリで複数のモジュールをサポートします](https://github.com/golang/go/wiki/Modules#faqs--multi-module-repositories)が、一般的には1モジュール/リポジトリより継続敵に管理するためにより多くのことをしなくてはいけなくなります。)
ようやくすると、repository/modules/package間の関係は、

- 1リポジトリで1つ以上のGoモジュールを含む。
- いずれのモジュールも1つ以上のGoパッケージを含む。
- いずれのパッケージも1つのディレクトリ内で1つ以上のGoソースから成る。

モジュールはセマンティックバージョン体系でなければいけません。通常は`v(major).(minor).(patch)`の形で`v0.1.0`, `v1.2.3`, `v1.5.0-rc.1`のように採番する。先頭の`v`は必須です。
もしGitを使っている場合、これらのバージョンを用いて[tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging)リリースされます。Publicとprivateモジュールリポジトリ、プロキシは使用可能になります。[FAQ:Are there "always on" module repositories and enterprise proxies?](https://github.com/golang/go/wiki/Modules#are-there-always-on-module-repositories-and-enterprise-proxies)

### go.mod

moduleはGoソースのルートディレクトリにある`go.mod`ファイルとGoソースのツリー構成によって決められます。
moduleのソースコードはGOPATHの外に配置されているかもしれません。`go.mod`は4つのディレクティブ`module`, `require`, `replace`, `exclude`から構成されます。

モジュール`github.com/my/thing`に対する`go.mod`の例を挙げます。

```go
module github.com/my/thing

require (
    github.com/some/dependency v1.2.3
    github.com/another/dependency/v4 v4.0.0
)
```

モジュールは`go.mod`の`module`ディレクティブでmodule pathを用いて自身を宣言します。module内の全パッケージのimport pathは共通のプレフィックスとしてmodule pathを共有します。moduleパスとパッケージディレクトリまでの`go.mod`からの相対パスはpackageのimportパスを決定します。

例えば、インポートパスが`github.com/user/mymod/foo`と`github.com/user/mymod/bar`の2つのパッケージを含む`github.com/user/mymod`モジュールをリポジトリに作っているとして、一般的にその時の`go.mod`の1行目ではモジュールパスとして`module github.com/user/mymod`を宣言して、対応する構成は以下のようになっているでしょう。

```bash
mymod
|-- bar
|   `-- bar.go
|-- foo
|   `-- foo.go
`-- go.mod
```

Goソースの中で、パッケージはモジュールパスを含んだフルパスでインポートされ使用される。例えば、上記の例で言えば、`go.mod`の中で`module github.com/user/mymod/`とした場合、外部で使用する時には下記のようにすればいい。

```go
import "github.com/user/mymod/bar"
```

パッケージ`bar`がmodule `github.com/user/mymod`からインポートされる。

`exclude`と`replace`ディレクティブは現在のmainモジュールにのみ動作します。mainモジュールをビルドする時、モジュール内の`exclude`と`replace`ディレクティブはmainモジュール以外では無視されます。したがって、`exluce`と`replace`はmainモジュールを、依存関係により制御されることなく、完全に制御できます。[detail:When should i use replace directice](https://github.com/golang/go/wiki/Modules#when-should-i-use-the-replace-directive)


### Version Selection
