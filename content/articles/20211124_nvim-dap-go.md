---
title: Neovim + DAP + Go
tags: ["neovim", "dap", "go", "debug"]
date: 2021-11-24
published: true
---

## TL;DR

[DAP(Debug Adapter Protocol)](https://microsoft.github.io/debug-adapter-protocol/)の力を借りれば、Neovim上でも視覚的にGoのデバッグが出来ました。
![nvim-dap-go](https://lunarxlark.github.io/gif/nvim-dap-go.gif)

ここでは下記については話しません。
- DAPの詳細
- packer.nvimでのインストール方法
- vscode-goのdebugAdapter.jsの使用方法


## Neovimでデバッグするモチベーション

v0.2.2から色々変わったNeovimを久々に触ってみようかと思いinit.luaへ以降して遊んでいました。
起動が早いだけでなく、LSP系の動作もサクサク動いてここ2ヶ月程快適に過ごしています。

ただ、VSCodeでのデバッグ機能がとても便利なため、デバッガ使用時のみこの快適な環境を離れていました。

どうにかNeovim上で完結させられないか...。


## [DAP](https://microsoft.github.io/debug-adapter-protocol/)

DAPの説明はリンク先や他HPをご参照ください。

イメージを持ってもらうため[LSP](https://microsoft.github.io/language-server-protocol/)を引き合いに出すと、LSPがテキスト編集時に定義や変更内容や補完候補等を返すのに対して、DAPはデバッガとやりとりしてデバッグ情報を返す位のイメージを持ってもらえれば大丈夫だと思います。


## Neovim + DAP + Go

使うプラグインは下記になります。
- [mfussenegger/nvim-dap](https://github.com/mfussenegger/nvim-dap) : NeovimのDAPクライアント。VSCodeのlaunch.jsonも使用可
- [rcarriga/nvim-dap-ui](https://github.com/rcarriga/nvim-dap-ui) : nvim-dapを視覚的に操作しやすいUI
- [leoluz/nvim-dap-go](https://github.com/leoluz/nvim-dap-go) : DAPのGo用設定(delve dap起動)
- [nvim-treesitter/nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) nvim-dap-goでテストをデバッグする際、近くのテストを見つけるのに必要

### packer.nvimの設定

VSCodeのlaunch.jsonを使用しない場合、`require('dap.ext.vscode').load_launchjs()`は不要です。
launch.jsonを読み込めた場合、デバッグ方法の選択時にlaunch.jsonの内容が追加表示されて選択可能になります。

```lua
-- dap plugin installation with packer.nvim
...
        -- dap
        use {
            "rcarriga/nvim-dap-ui",
            requires = {
                "mfussenegger/nvim-dap",
                "leoluz/nvim-dap-go",
                "nvim-treesitter/nvim-treesitter"
            },
            config = function ()
                vim.fn.sign_define('DapBreakpoint', {text='⛔', texthl='', linehl='', numhl=''})
                vim.fn.sign_define('DapStopped', {text='👉', texthl='', linehl='', numhl=''})
                require('dapui').setup()
                require('dap-go').setup()
                require('dap.ext.vscode').load_launchjs()
            end
        }
...
```


### GoでDAPを起動する際の設定

Goの場合、vscode-goのdebugAdapter.jsを使用することも出来ます。ここでは、delveを待ち構えさせます。
```lua
local dap = require("dap")
dap.adapters.go = function(callback, config)
    local stdout = vim.loop.new_pipe(false)
    local handle
    local pid_or_err
    local port = 38697
    local opts = {
      stdio = {nil, stdout},
      args = {"dap", "-l", "127.0.0.1:" .. port},
      detached = true
    }
    handle, pid_or_err = vim.loop.spawn("dlv", opts, function(code)
      stdout:close()
      handle:close()
      if code ~= 0 then
        print('dlv exited with code', code)
      end
    end)
    assert(handle, 'Error running dlv: ' .. tostring(pid_or_err))
    stdout:read_start(function(err, chunk)
      assert(not err, err)
      if chunk then
        vim.schedule(function()
          require('dap.repl').append(chunk)
        end)
      end
    end)
    -- Wait for delve to start
    vim.defer_fn(
      function()
        callback({type = "server", host = "127.0.0.1", port = port})
      end,
      100)
end
```

### デバッグの起動設定

VSCodeのlaunch.jsonに該当する設定になります。デバッグされるプログラム起動方法になります。
VSCode同様、`relativeFileDirname`や`workspaceFolder`等変数が用意されています。詳細は`:h dap.txt`を参照してください。
```lua
 dap.configurations.go = {
    {
        type = "go",
        name = "Debug",
        request = "launch",
        program = "${file}"
    },
    {
        type = "go",
        name = "Debug test", -- configuration for debugging test files
        request = "launch",
        mode = "test",
        program = "${file}"
    },
    -- works with go.mod packages and sub packages
    {
        type = "go",
        name = "Debug test (go.mod)",
        request = "launch",
        mode = "test",
        program = "./${relativeFileDirname}"
    }
}
```

最後にVSCode風のキーマップです。
```lua
local function map(mode, lhs, rhs, opts)
    local options = {noremap = true}
    if opts then options = vim.tbl_extend('force', options, opts) end
    vim.api.nvim_set_keymap(mode, lhs, rhs, options)
end

map("n", "<leader>d", ":lua require'dapui'.toggle()<CR>", { silent = true})
map("n", "<leader><leader>df", ":lua require'dapui'.eval()<CR>", { silent = true})
map("n", "<F5>", ":lua require'dap'.continue()<CR>", { silent = true})
map("n", "<F10>", ":lua require'dap'.step_over()<CR>", { silent = true})
map("n", "<F11>", ":lua require'dap'.step_into()<CR>", { silent = true})
map("n", "<F12>", ":lua require'dap'.step_out()<CR>", { silent = true})
map("n", "<leader>b", ":lua require'dap'.toggle_breakpoint()<CR>", { silent = true})
map("n", "<leader>bc", ":lua require'dap'.set_breakpoint(vim.fn.input('Breakpoint condition: '))<CR>", { silent = true})
map("n", "<leader>l", ":lua require'dap'.set_breakpoint(nil, nil, vim.fn.input('Log point message: '))<CR>", { silent = true})
```


## まとめ

上記設定は、各プラグインのREADMEやWikiに書いてある設定とほぼ同じです。

あまり小難しい設定することなく動きますし、READMEやWikiが充実しているので試してみてはいかがでしょうか。

それではよきVim/Neovimライフを。
