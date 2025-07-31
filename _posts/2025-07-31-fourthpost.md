---
title: "vim-slimeでREPL"
layout: post
---
tags: [oil.nvim, neovim, file explorer, plugin]

# vimのマーク機能について
m[alphabet]で場所をマーク。大文字だと、他のセッションからでもそのファイルにアクセス可能。
`[alphabet]`でマークした場所にジャンプ。便利なので、init.luaをmVした。
このブログもmPした

# vimのread,writeコマンドについて
:r(read) !lsなど、外部コマンドの出力を現在のカーソル位置に挿入。
:write !sh などバッファの内容を標準入力としてコマンドにわたす。


# コマンドラインウィンドウ
q: で過去コマンドを表示、そのままそこで編集することも可能である。


どれも非常に便利な機能と思う。
