---
title : My Favorite Usage urfave/cli
tags : ["go"]
date : 2020-02-06
published : true
---

参加しているプロジェクトで`urfave/cli`を使っている。使い方は[Example](https://github.com/urfave/cli/blob/master/docs/v2/manual.md#examples)にあるのと同じ書き方で使っている。
大半のバッチ処理をコマンドとして記述しているので、Exampleの書き方だとだいぶ見辛くなってきた。縦長のコマンド定義とオプション説明で目的の処理を探すのにもページ送りを何度もする。なんとかしたい。

<!--more-->

golangを書き始めてそろそろ1年になり少し慣れてきたし、大好きなコマンド[ghq](https://github.com/x-motemen/ghq)がv1.0.0を迎えたりドメイン変更したのを機にソースを読み始めた。
この`ghq`も`urfave/cli`を使っているのだけど、読み始めてすぐになんて読みやすいんだ!と感激して、その書き方を踏まえてこう書いたらもっと読みやすくなるんじゃないかっていうのを残す。

まず、ghqのソース大まかに下記のようになっている。
main.goではentrypointとしてのみあり、実装するコマンドと各コマンドの実装も別ファイルに分けている。読みやすい!!

```go
// main.go
func main() {
	if err := newApp().Run(os.Args); err != nil {
		os.Exit(i1)
	}
}

func newApp() *cli.App {
	app := cli.NewApp()
	app.Name = "ghq"
	app.Commands = commands
	return app
}
```

```go
// commands.go
var commands = []*cli.Command{
	commandGet,
	commandList,
  ...
}

var commandGet = &cli.Command{
	Name:  "get",
	Action: doGet,
}
```

```go
# cmd_get.go
func doCreate(c *cli.Context) error {
  ...
}
```

今のままでも十分読みやすいんだけど、コマンドが増えていくと同じ階層に同じprefixのファイルも増える。そうなるとディレクトリを掘りたくなったので下記のようにしてみた。

```sh
x-motemen/ghq/
　├ cmd/
　│　└ common.go
　│　└ get.go
　│　└ list.go
　│　└ root.go
　├ repository/
　│　└ local.go
　│　└ remote.go
...
　├ main.go
　├ vcs.go
　└ url.go
```

`urfave/cli`のstruct階層とディレクトリ構成が似ていて、個人的にはとっつきやすいし読みやすいし、まとまっているのでこの書き方が好きだなって感じた。
他に`urfave/cli`を使っているコマンドも見ていこう。オプションの初期化も気になる...やりたいことは多いなぁ。
