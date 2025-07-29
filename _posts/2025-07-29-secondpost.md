---
title: "vim-slimeでREPL"
layout: post
---
tags: [oil.nvim, neovim, file explorer, plugin]
# vim(neovim)のREPL機能について
vim-slime＋tmuxを試す。普段の解析はR言語がメインのため、R.nvimという選択肢もある。iron.nvimもある。
しかし、他言語を今後学習する可能性も大いに有り得るためより普遍的な方法を志向した結果、この方針になった。

```
https://github.com/jpalardy/vim-slime.git

-- -----------------------------------------------------------------------------
-- 基本設定 (Options)
-- -----------------------------------------------------------------------------
vim.o.number = true             -- 行番 を表示
-- vim.o.relativenumber = true     -- 現在行を0として相対的な行番号を表示
vim.o.signcolumn = "yes"        -- 常にサインコラムを表示（lspの警告やgitの差分表示用）
vim.o.wrap = false              -- 長い行を折り返さない
vim.o.tabstop = 4               -- タブの幅をスペース4つ分に設定
vim.o.swapfile = false          -- スワップファイルを作成しない
vim.o.winborder = "rounded"     -- ウィンドウの境界線を角丸にする
vim.g.mapleader = " "           -- リーダーキーをスペースキーに設定
-- クリップボードの設定
vim.o.clipboard = "unnamedplus" -- システムクリップボードを使用

vim.g.slime_target = "tmux" -- slimeのターゲットをtmuxに設定
vim.g.slime_default_config = {
  socket_name = "/private/tmp/tmux-501/default", -- tmuxのソケット名をフルパスで指定
  target_pane = ":.+",      -- ターゲットペインを指定
  send_timeout = 0,         -- 送信タイムアウトを無効化
}
-- デフォルトのキーマップを無効化
vim.g.slime_no_mappings = 1
-- ターゲットを尋ねるプロンプトを無効化
vim.g.slime_dont_ask_default = 1

-- ▼▼▼ この一行を追加 ▼▼▼
-- 全てのファイルタイプで共通のセル区切り文字を設定
vim.g.slime_cell_delimiter = "###"
-- -----------------------------------------------------------------------------
-- k
-- プラグイン管理 (built-in package manager)
-- -----------------------------------------------------------------------------
vim.pack.add({
	{ src = "https://github.com/stevearc/oil.nvim" },
	{ src = "https://github.com/echasnovski/mini.pick" },
	{ src = "https://github.com/neovim/nvim-lspconfig" },
	{ src = "https://github.com/chomosuke/typst-preview.nvim" },
	{ src = "https://github.com/github/copilot.vim" },
	{ src = "https://github.com/copilotc-nvim/copilotchat.nvim" },
	{ src = "https://github.com/nvim-lua/plenary.nvim" },
	{ src = "https://github.com/jpalardy/vim-slime" },
})

-- -----------------------------------------------------------------------------
-- 各機能の設定とキーマップ (configurations & keymaps)
-- -----------------------------------------------------------------------------

-- プラグインの設定
require("mini.pick").setup()
-- require("oil").setup()
-- oil.nvimの設定
require('oil').setup({
    keymaps = {
        ['yp'] = {
            desc = 'Copy filepath to system clipboard',
            callback = function()
                require('oil.actions').copy_entry_path.callback()
                -- 選択されたエントリのパスをシステムクリップボードにコピー
                vim.fn.setreg("+", vim.fn.getreg(vim.v.register))
            end,
        },
    },
    -- その他のoil.nvimの設定があればここに追加
})
require("copilotchat").setup()

-- tmuxとの連携
-- vim.g.slime_tmux_target = ":.1" 
-- LSP (Language Server Protocol) の設定
-- vim.lsp.enable({ "lua_ls" })
vim.lsp.enable({ "lua_ls", "r_language_server", "bashls" })

-- キーマップ
local keymap = vim.keymap.set
local opts = { noremap = true, silent = true }
-- ▼▼▼ vim-slime のキーマップ (確実な代替案) ▼▼▼

-- -- 「カーソル行を送信して一行下に移動する」を安全に行う関数
-- local function send_and_move_down()
--   vim.api.nvim_command('normal! <Plug>SlimeLineSend')
--   vim.api.nvim_command('normal! j')
-- end

-- 【メイン】<leader>e で現在の行または選択範囲を送信
keymap("n", "<leader>l", "<Plug>SlimeLineSend", opts)
keymap("x", "<leader>l", "<Plug>SlimeRegionSend", opts)

-- 現在の段落(paragraph)を送信
keymap("n", "<leader>p", "vip<Plug>SlimeRegionSend", opts)

-- 現在の「セル」を送信
keymap("n", "<leader>c", "<Plug>SlimeSendCell", opts)


--
--
-- 基本操作
keymap("n", "<leader>w", ":write<CR>", opts)
keymap("n", "<leader>q", ":quit<CR>", opts)
keymap("n", "<leader>z", ":update<CR> :source %<CR>", opts)

-- プラグイン操作
keymap("n", "<leader>f", ":Pick files<CR>", opts)
keymap("n", "<leader>h", ":Pick help<CR>", opts)
keymap("n", "<leader>o", ":Oil<CR>", opts)

-- jkでのノーマルモードへの戻り
keymap("i", "jk", "<Esc>", opts)

-- ctrl + hjklでのウィンドウ移動
keymap("n", "<C-h>", "<C-w>h", opts)
keymap("n", "<C-j>", "<C-w>j", opts)
keymap("n", "<C-k>", "<C-w>k", opts)
keymap("n", "<C-l>", "<C-w>l", opts)

-- LSP操作
keymap("n", "<leader>lf", vim.lsp.buf.format, opts)

-- -----------------------------------------------------------------------------
-- カラースキーム (Colorscheme)
-- -----------------------------------------------------------------------------
vim.cmd("colorscheme minischeme")
vim.cmd("hi StatusLine guibg=NONE")
```
こんな感じに設定。意外と<leader>pで段落、<leader>cでセルを送信できるのが便利である。
最初からこれを意識してコードを執筆するため、以前よりも見た目はスッキリしたような・・・？
tmuxのペインに送る所の設定でやや手間取ったが、無事に動作している。
