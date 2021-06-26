+++
title = "From denite.nvim To fzf.vim"
date = 2020-02-11T15:57:07+09:00
tags = ["vim"]
draft = false
+++

> denite.nvimとfzf.vimって比較記事?

違います。断捨離した結果、fzf.vimで事足りてしまったという記事です。

denite.nvimとfzf.vimは、一見やれることが似ているように見えますが提供しているインタフェースが違います。
denite.nvimの方が拡張性/汎用性が高いです。Pythonスクリプトを呼び出せますし。

> どうしてやめたん？

Python3とpipの環境整備に疲れたというのが理由で完全に力不足なだけです。

そもそも使いこなせていなかったっていうのも大きい。自分に必要な機能が何か見直したら次のがあれば十分っぽい。

- コマンドの結果の一覧表示([x-motemen/ghq](https://github.com/x-motemen/ghq) list, [mattn/memo](https://github.com/mattn/memo) list, history等)
- 一覧表示の後のアクションを指定可能(cd, vim)
- buffer切替

[fzf.vim](https://github.com/junegunn/fzf.vim)は[fzf](https://github.com/junegunn/fzf)のついでに入れていただけで全く使っていなかった。[Shougo/denite.nvim](https://github.com/Shougo/denite.nvim)でfzf.vimで同じことが出来るし、sourceの拡張がいくつもあるのでそれで十分だった。

もともとfzfが好きなのもあって、fzf.vimで上記が実現出来るように設定した、っていうかhelpからパクってきた。

何日か使っていて快適に使えているので結構満足。previewは[bat](https://github.com/sharkdp/bat)ってコマンド入れないとsyntax highlightされなかったのでいれたけど、なんとなくそっちの方がリッチっぽいという理由だけなので重ければそのうち消す。

```vim
command! -bang -nargs=? -complete=dir Files call fzf#vim#files(
  \ <q-args>,
  \ fzf#vim#with_preview(),
  \ <bang>0)

command! -nargs=0 Ghq call fzf#run({
  \ 'source' : 'ghq list --full-path',
  \ 'sink' : 'cd'
  \})

command! -nargs=0 Mru call fzf#run({
  \ 'source' : v:oldfiles,
  \ 'sink' : 'edit',
  \ 'options' : '-m -x +s',
  \ 'down' : '40%'
  \})

command! -bang -nargs=? Memo call fzf#vim#files(
  \ expand($HOME . '/.config/memo/_posts'),
  \ fzf#vim#with_preview({'options': ['--layout=reverse', '--info=inline']}),
  \ <bang>0)

command! -bang -nargs=* Rg call fzf#vim#grep(
  \ 'rg --column --no-heading --color=always --smart-case '.shellescape(<q-args>),
  \ 1,
  \ fzf#vim#with_preview(), <bang>0)

nmap ,f :Files<CR>
nmap ,s :Snippet<CR>
nmap ,b :Buffers<CR>
nmap ,g :Ghq<CR>
nmap ,m :Mru<CR>
nmap ,mm :Memo<CR>
nmap ,rg :Rg<CR>
```

