---
title: 2021年版 vim + goplsの設定
date: 2021-06-16T08:53:28+09:00
tags: ["vim", "go", "gopls"]
published: true
---

goplsが出てから, vimでも定義ジャンプやシンボル検索、ドキュメント参照等が行えるようになった。

たまにVSCodeを触りvimでの作業を改善できないか考える中で、自身の設定が古いことに気付いた。また、ググってもなかなか出てこなかったのでメモとして記述する。

cf. [GitHub dotfiles](https://github.com/lunarxlark/dotfiles)


いきなりだが、vimrcとvim-lsp-settings/settings.jsonを抜粋して貼り付ける。

以前、GoではLspCodeAction, LspCodeLens等をサポートしていなかったが、今では使えるようになっている。

キーマップに設定している関数は全てGoで使用出来る。

ただし、カーソルがどこにいても実行出来るわけではないので注意が必要。

LspCodeActionはカーソルの位置によって実行内容が変わるのでそこも注意。

```vim
...
Plug 'prabirshrestha/vim-lsp'
Plug 'mattn/vim-lsp-settings'
Plug 'prabirshrestha/asyncomplete-lsp.vim'
Plug 'mattn/vim-gomod'
...

" ------------------------------------------------------------------------------
" vim-lsp
" ------------------------------------------------------------------------------
function! s:on_lsp_buffer_enabled() abort
    setlocal omnifunc=lsp#complete
    setlocal signcolumn=yes
    if exists('+tagfunc') | setlocal tagfunc=lsp#tagfunc | endif
    nmap <buffer> <leader>ac <plug>(lsp-code-action)
    nmap <buffer> <leader>cl <plug>(lsp-code-lens)
    nmap <buffer> <leader>df <plug>(lsp-definition)
    nmap <buffer> <leader>dd <plug>(lsp-document-diagnostics)
    nmap <buffer> <leader>im <plug>(lsp-implementation)
    nmap <buffer> <leader>pdf <plug>(lsp-peek-definition)
    nmap <buffer> <leader>sm <plug>(lsp-document-symbol-search)
    nmap <buffer> <leader>Sm <plug>(lsp-workspace-symbol-search)
    nmap <buffer> <leader>rf <plug>(lsp-references)
    nmap <buffer> <leader>td <plug>(lsp-type-definition)
    nmap <buffer> <leader>rn <plug>(lsp-rename)
    nmap <buffer> <leader>en <plug>(lsp-next-error)
    nmap <buffer> <leader>ep <plug>(lsp-previous-error)
    nmap <buffer> <leader>ho <plug>(lsp-hover)

    let g:lsp_format_sync_timeout = 500
    autocmd! BufWritePre *.rs,*.go call execute('LspDocumentFormatSync')

    " refer to doc to add more commands
endfunction

augroup lsp_install
    au!
    " call s:on_lsp_buffer_enabled only for languages that has the server registered.
    autocmd User lsp_buffer_enabled call s:on_lsp_buffer_enabled()
augroup END
```

```json
// vim-lsp-settings/settings.json
{
  "go": {
    "gopls": {
      "initialization_options": {
        "analyses" : {"fillstruct":true},
        "staticcheck": true,
        "directoryFilters": [
          "-debug"
        ],
        "completeUnimported": true, "usePlaceholders": true,
        "matcher": "fuzzy",
        "codelenses": {
          "gc_details": false,
          "generate": true,
          "test": true,
          "tidy": true,
          "vendor": false
        },
        "hoverKind": "SynopsisDocumentation"
      }
    }
  }
}
```

## vim + goplsで新しく設定したこと

### LspCodeAction

vim-lspのhelpにあるように、LspCodeActionが実行出来る行では `A>` が表示される。

Goで出来るCodeAction全体はわからないが、今のところは下記が出来ることを確認した。

- structを初期値で補完 (要設定: `"analyses": {"fillstruct": true}`)
- returnで関数フォーマットを補完


```vim
":help vim-lsp.txt
g:lsp_document_code_action_signs_enabled
                               *g:lsp_document_code_action_signs_enabled*
...
    `LspCodeActionText` defaults to `A>`.
...
```

![LspCodeAction](/images/screenshot_20210616-093637.png)


### LspDocumentDiagnostics

(これは以前から設定できたみたい。)

golintがdeprecatedになったので、linterにstaticcheckを設定したかった。

また、VSCodeのタブ `PROBLEMS` のようにコードでの問題箇所を確認したかった。

(staticcheckのどのケースにひっかかったのかがわからない...。AXXXも出力したい。)

![LspDocDiagnostics](/images/screenshot_20210616-100337.png)


### LspCodeLens

vim-lsp-settings/settings.jsonにある機能がファイルに応じて出来るようになる。

- `_test.go` -> `go test`

![LspCodeLens-test](/images/screenshot_20210616-093830.png)


- `go.mod` -> `go tidy` (go.modのfiletypeをgo or gomodにする必要がある。mattn/vim-gomodを入れるのが無難。)

![LspCodeLens-tidy](/images/screenshot_20210616-095605.png)


```vim
":help vim-lsp.txt
LspCodeLens                                                   *:LspCodeLens*

Gets a list of possible commands that can be executed on the current document.
```


## まとめ

2021年になり、goplsはとても使い心地よく改善されてきました。

まだ手の届かないところがありますが、たくさんのプラグインを入れることなくgoplsで出来るだけ済むように設定することでメンテしやすくなったと思います。
