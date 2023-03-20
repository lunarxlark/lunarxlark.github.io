---
title : Error 'undefined' when go run
tags : ["go"]
date : 2020-02-04
published : true
---

x…motemen/ghqを写経している時、 `go run main.go` 出来ないことに気付いた。
下記が実行時のエラーになる。ちなみに、 `go build` は出来る。

```sh
~/d/s/g/x/ghq  >>> go run main.go
# command-line-arguments
./main.go:38:17: undefined: commands
~/d/s/g/x/ghq  >>>
```

`commands` が見つからない？同じ階層の `commands.go` には下記記述がちゃんとあるのに、どうして見つからない...。

```go
var commands = []*cli.Command{
	commandGet,
	commandList,
	commandRoot,
	commandCreate,
}
```

理由は、`go run`の引数に指定したファイル(+ importされるパッケージ)しか読み込まないから。

実際、`go run main.go commands.go`に変更すると上記エラーは解消される...が、別のエラーとなる。
今度は違うのが見つからないって言われる。

```sh
~/d/s/g/x/ghq master >>> go run main.go commands.go
# command-line-arguments
./commands.go:25:10: undefined: doGet
./commands.go:50:10: undefined: doList
./commands.go:62:10: undefined: doRoot
./commands.go:71:10: undefined: doCreate
```

同じ階層で必要なファイルを引数に全て指定すればいいのだけど、そんなことはしたくない。
正規表現で指定してもいいけど、`_test.go`があると`go run`で実行出来ないってエラーになる。

```sh
~/d/s/g/x/ghq master >>> go run *.go
go run: cannot run *_test.go files (cmd_create_test.go)
```

正規表現`(?!._test).go`でファイル指定も出来ない。shellには書きたくない。

どうすれば...答えは `go run .`

```sh
~/d/s/g/x/ghq master >>> go run .
NAME:
   ghq - Manage remote repository clones
...
```


パッとググっても複数ファイル指定の記事ばかり出てきたし、下記stackoverflowのvote数も低かったので迷った...

[stackoverflow: go build works fine but go run fails](https://stackoverflow.com/questions/21293000/go-build-works-fine-but-go-run-fails)
