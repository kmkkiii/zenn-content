---
title: "カスタマイズの基本"
---

## MiniMaxの設定は「自分のもの」

この章がMiniMaxを選ぶ最大の理由を体感できる章や。

MiniMaxがセットアップ時に生成したファイルは、あなたのものや。`~/.config/nvim-minimax/plugin/`以下のファイルを直接編集することで、自分好みに設定を変えられる。

各ファイルにはコメントが丁寧に書いてあって、それ自体が学習教材になっとる。まず設定ファイルを開いて、コメントを読んでみよう：

```vim
" MiniMaxを起動した状態で
:edit ~/.config/nvim-minimax/plugin/10_options.lua
```

## オプションの変更（10_options.lua）

`vim.o`（または`vim.opt`）を使ってNeovimの動作を設定する：

```lua
-- plugin/10_options.lua

-- 行番号
vim.o.number = true           -- 絶対行番号を表示
vim.o.relativenumber = true   -- 相対行番号を表示（両方trueで相対+現在行は絶対）

-- インデント
vim.o.expandtab = true        -- タブをスペースに展開
vim.o.tabstop = 2             -- タブの表示幅
vim.o.shiftwidth = 2          -- インデント幅

-- 表示
vim.o.wrap = false            -- 折り返さない
vim.o.scrolloff = 8           -- カーソルを画面端から8行離す
vim.o.sidescrolloff = 8       -- 横スクロールのオフセット
vim.o.colorcolumn = '80'      -- 80文字目に縦線を表示

-- 検索
vim.o.ignorecase = true       -- 検索で大文字小文字を無視
vim.o.smartcase = true        -- 大文字を含む場合は区別する

-- ファイル
vim.o.swapfile = false        -- スワップファイルを作らない
vim.o.undofile = true         -- Undoをファイルに永続化

-- クリップボード
vim.o.clipboard = 'unnamedplus'  -- システムクリップボードと共有
```

:::message
設定を変更したら、`:source %`でファイルを再読み込みするか、Neovimを再起動すると設定が反映される。
:::

## キーマップの追加（20_keymaps.lua）

`vim.keymap.set()`でカスタムキーマップを追加する：

```lua
-- vim.keymap.set({mode}, {lhs}, {rhs}, {opts})
-- mode: 'n'=ノーマル, 'i'=インサート, 'v'=ビジュアル, 'x'=ビジュアルのみ

local map = vim.keymap.set

-- 便利なキーマップの例

-- Escの代わりにjkで抜ける（好みが分かれる）
map('i', 'jk', '<Esc>', { desc = 'Exit insert mode' })

-- 選択範囲を維持してインデント
map('v', '<', '<gv', { desc = 'Indent left' })
map('v', '>', '>gv', { desc = 'Indent right' })

-- ウィンドウ移動を簡単に
map('n', '<C-h>', '<C-w>h', { desc = 'Move to left window' })
map('n', '<C-j>', '<C-w>j', { desc = 'Move to down window' })
map('n', '<C-k>', '<C-w>k', { desc = 'Move to up window' })
map('n', '<C-l>', '<C-w>l', { desc = 'Move to right window' })

-- バッファを素早く移動
map('n', '<Tab>', ':bnext<CR>', { desc = 'Next buffer' })
map('n', '<S-Tab>', ':bprev<CR>', { desc = 'Previous buffer' })

-- 検索ハイライトを消す
map('n', '<Leader>nh', ':nohlsearch<CR>', { desc = 'Clear search highlight' })
```

### mini.clueへの登録

カスタムキーマップを追加した時、`mini.clue`のヒントにも登録しておくと覚えやすい：

```lua
-- plugin/30_mini.lua のmini.clue設定に追記
require('mini.clue').setup({
  -- ... 既存の設定 ...
  clues = {
    -- カスタムグループの追加
    { mode = 'n', keys = '<Leader>n', desc = 'No...' },
  },
})
```

## カラースキームの変更

MiniMaxは `mini.nvim` 付属のカラースキームを使っとる。変更する方法はいくつかある：

### mini.nvim付属のカラースキームを使う

```vim
:colorscheme minischeme   " MiniMaxデフォルト
:colorscheme minicyan
:colorscheme minidark
```

### 外部カラースキームを追加する

`plugin/40_plugins.lua`にプラグインを追加する。Neovim 0.12以降は組み込みの`vim.pack`、0.11以前は`mini.deps`を使う：

```lua
-- plugin/40_plugins.lua に追記（Neovim 0.12 / vim.packの場合）

-- everforest（落ち着いた緑系）
vim.pack.add('https://github.com/sainnhe/everforest')
vim.cmd('colorscheme everforest')

-- tokyonight（人気のモダンなテーマ）
vim.pack.add('https://github.com/folke/tokyonight.nvim')
require('tokyonight').setup({ style = 'moon' })
vim.cmd('colorscheme tokyonight-moon')

-- catppuccin（パステル系でおしゃれ）
vim.pack.add('https://github.com/catppuccin/nvim')
require('catppuccin').setup({ flavour = 'mocha' })
vim.cmd('colorscheme catppuccin')
```

追記したら`:PackInstall`でインストール：

```vim
:PackInstall
```

## mini.nvimモジュールの設定変更

各mini.nvimモジュールは`setup()`関数にテーブルを渡して設定できる。`plugin/30_mini.lua`を開いて確認してみよう：

```lua
-- mini.filesの設定例
require('mini.files').setup({
  windows = {
    preview = true,      -- プレビューを有効化
    width_focus = 30,    -- フォーカス時のウィンドウ幅
    width_preview = 50,  -- プレビューウィンドウの幅
  },
  options = {
    use_as_default_explorer = true,  -- netrwの代わりに使う
  },
})

-- mini.statuslineの設定例
require('mini.statusline').setup({
  use_icons = true,
  set_vim_settings = true,
})
```

各モジュールの詳細はNeovimのヘルプで確認できる：

```vim
:h mini.files
:h mini.pick
:h mini.completion
:h mini.git
```

## ファイルタイプ固有の設定（after/ftplugin/）

特定の言語ファイルでだけ有効な設定は `after/ftplugin/` に書く：

```lua
-- after/ftplugin/python.lua
vim.opt_local.tabstop = 4      -- Pythonは4スペース
vim.opt_local.shiftwidth = 4
vim.opt_local.colorcolumn = '88'  -- blackのデフォルト行長に合わせる
```

```lua
-- after/ftplugin/markdown.lua
vim.opt_local.wrap = true       -- Markdownは折り返す
vim.opt_local.spell = true      -- スペルチェックを有効化
vim.opt_local.textwidth = 80    -- 80文字で自動改行
```

## プラグインを追加する

Neovimのバージョンによって使うパッケージマネージャが異なる。

:::message
**Neovim 0.12以降**: 組み込みの`vim.pack`を使う（MiniMaxの0.12設定が採用）
**Neovim 0.10/0.11**: `mini.deps`を使う（MiniMaxの旧設定）
:::

### vim.pack（Neovim 0.12+、推奨）

Neovim 0.12からは`vim.pack`がコアに組み込まれた。外部のパッケージマネージャプラグインが不要になる：

```lua
-- plugin/40_plugins.lua に追記

-- シンプルなプラグイン（GitHubのURLをフルで指定）
vim.pack.add('https://github.com/tpope/vim-fugitive')

-- オプション付き（バージョン指定、フック等）
vim.pack.add({
  src = 'https://github.com/zbirenbaum/copilot.lua',
  version = '*',   -- 最新の安定版
})
```

プラグインの管理コマンド：

```vim
:PackInstall   " プラグインをインストール
:PackUpdate    " プラグインを更新
:PackClean     " 不要なプラグインを削除
```

### mini.deps（Neovim 0.10/0.11）

Neovim 0.10/0.11を使っている場合はMiniMaxのパッケージマネージャである`mini.deps`を使う：

```lua
-- シンプルなプラグイン
MiniDeps.add({ source = 'tpope/vim-fugitive' })

-- 遅延読み込み
MiniDeps.later(function()
  MiniDeps.add({ source = 'folke/sidekick.nvim' })
  require('sidekick').setup({})
end)
```

```vim
" 更新
:lua MiniDeps.update()
```

:::message
`mini.deps`はNeovim 0.12リリースに合わせて開発終了（メンテナンスのみ）予定。Neovim 0.12に上げたタイミングで`vim.pack`に移行するのがおすすめや。
:::

## 設定のバックアップとGit管理

設定ファイルをGitで管理しとくと、変更履歴が残って安心やで：

```bash
cd ~/.config/nvim-minimax

# すでにGitリポジトリにしている場合
git add plugin/10_options.lua
git commit -m "feat: adjust tab settings for Python"

# 設定を誤って壊した場合
git diff                    # 変更内容を確認
git checkout -- plugin/10_options.lua  # 元に戻す
```

## まとめ：カスタマイズの優先順位

初心者が最初にカスタマイズすべきこと：

1. **オプション**: `10_options.lua` でインデント幅、行番号表示を好みに合わせる
2. **カラースキーム**: 目に優しいテーマを設定して作業しやすくする
3. **キーマップ**: 頻繁に使う操作に短いキーを割り当てる
4. **言語固有設定**: `after/ftplugin/`で言語ごとのルール（インデント等）を設定

少しずつカスタマイズして、自分だけの最高の環境を作り上げていこう！

次はボーナスチャプターで、AIコーディングプラグインの導入を解説するで！
