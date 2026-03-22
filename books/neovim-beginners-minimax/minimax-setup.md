---
title: "MiniMaxのセットアップ"
---

## MiniMaxをインストールする

準備が整ったところで、いよいよMiniMaxをセットアップしよう。

### 1. MiniMaxのクローンと設定生成

以下のコマンドを順番に実行してな：

```bash
# MiniMaxをクローン（任意のディレクトリでOK）
git clone --filter=blob:none https://github.com/nvim-mini/MiniMax ./MiniMax

# セットアップスクリプトを実行
NVIM_APPNAME=nvim-minimax nvim -l ./MiniMax/setup.lua
```

`setup.lua`を実行すると、あなたのNeovimのバージョンを自動で検出して、対応する設定ファイルを `~/.config/nvim-minimax/` に生成する。

出力例（Neovim 0.11の場合）：

```
Detected Neovim version: 0.11
Copying configuration for Neovim 0.11...
✓ Copied: init.lua
✓ Copied: plugin/10_options.lua
✓ Copied: plugin/20_keymaps.lua
✓ Copied: plugin/30_mini.lua
✓ Copied: plugin/40_plugins.lua
✓ Done! Configuration generated at ~/.config/nvim-minimax
```

### 2. 初回起動

```bash
NVIM_APPNAME=nvim-minimax nvim
```

:::message
エイリアスを設定した場合は `nvm` でもOKや。
:::

初回起動時は、プラグインの自動インストールが始まる。画面下部に進捗が表示されるので、完了するまで待ってな。

```
[mini.deps] Installing dependencies...
  ✓ nvim-treesitter/nvim-treesitter
  ✓ nvim-treesitter/nvim-treesitter-textobjects
  ✓ neovim/nvim-lspconfig
  ✓ stevearc/conform.nvim
  ✓ rafamadriz/friendly-snippets
[mini.deps] Done!
```

全部インストールされたら、`mini.starter`（スタート画面）が表示される。

### 3. スタート画面の操作

MiniMaxを起動するとこんな画面が表示される：

```
███╗   ███╗██╗███╗   ██╗██╗
████╗ ████║██║████╗  ██║██║
██╔████╔██║██║██╔██╗ ██║██║
██║╚██╔╝██║██║██║╚██╗██║██║
██║ ╚═╝ ██║██║██║ ╚████║██║

  Recent files
  > src/main.rs          e
  > README.md            e

  Built-in actions
  > New file             n
  > Quit                 q
```

キーボードの`e`や`n`を押すと対応するアクションが実行される。`q`で終了。

## 生成された設定ファイルの構造

`~/.config/nvim-minimax/` の中を見てみよう：

```bash
ls -la ~/.config/nvim-minimax/
```

```
~/.config/nvim-minimax/
├── init.lua                  # エントリーポイント・ヘルパー関数定義
├── plugin/
│   ├── 10_options.lua        # Neovimオプション設定
│   ├── 20_keymaps.lua        # キーマップ設定
│   ├── 30_mini.lua           # mini.nvimモジュール設定
│   └── 40_plugins.lua        # 非miniプラグイン設定
├── after/
│   ├── ftplugin/             # ファイルタイプ固有の設定
│   │   └── markdown.lua
│   ├── lsp/                  # 言語サーバーごとの設定
│   │   └── lua_ls.lua
│   └── snippets/             # 言語ごとのスニペット
│       └── lua.json
├── snippets/
│   └── global.json           # グローバルスニペット
└── nvim-pack-lock.json       # プラグインのバージョンロックファイル
```

各ファイルの役割を簡単に説明するで。

### init.lua

設定全体のエントリーポイント。`plugin/`ディレクトリ内のファイルが`10_`→`20_`→`30_`→`40_`の順で自動的に読み込まれる仕組みになっとる。

グローバルな`Config`テーブルとヘルパー関数も定義しとる：

- `Config.now()` — 起動時にすぐ実行する処理（UI関連等）
- `Config.later()` — 起動後に非同期で遅延実行する処理（頻繁に使わない機能等）
- `Config.on_event()` — 特定イベント時に実行
- `Config.on_filetype()` — 特定ファイルタイプ時に実行

`30_mini.lua`や`40_plugins.lua`でこのパターンを使ってプラグインの読み込みタイミングを制御している。

### plugin/10_options.lua

Neovim自体の動作設定。インデントのサイズ、行番号の表示方法、クリップボードの設定などが書いてある。

```lua
-- 例（実際のファイルから抜粋）
vim.o.number = true          -- 行番号を表示
vim.o.relativenumber = true  -- 相対行番号を表示
vim.o.expandtab = true       -- タブをスペースに変換
vim.o.tabstop = 2            -- タブ幅を2に設定
vim.o.shiftwidth = 2         -- インデント幅を2に設定
```

### plugin/20_keymaps.lua

カスタムキーマップの定義。Leaderキー（`<Space>`）を使ったショートカットが多数定義されとる。

```lua
-- 例
vim.g.mapleader = ' '  -- Leaderキーをスペースに設定
vim.keymap.set('n', '<Leader>ff', '<cmd>Pick files<cr>', { desc = 'Find files' })
```

### plugin/30_mini.lua

mini.nvimの各モジュール設定。このファイルが一番長く、MiniMaxの核心部分や。35個のmini.nvimモジュールの設定が書いてある。

### plugin/40_plugins.lua

mini.nvim以外のプラグイン（nvim-treesitter、nvim-lspconfig等）の設定。

## mini.nvimモジュール一覧

MiniMaxに含まれる35個のmini.nvimモジュールを機能カテゴリ別にまとめたで：

### UI・見た目系

| モジュール | 機能 |
|---|---|
| mini.starter | スタート画面 |
| mini.statusline | ステータスライン（画面下部の情報バー） |
| mini.tabline | タブライン（バッファ一覧） |
| mini.notify | 通知表示 |
| mini.icons | アイコン表示 |
| mini.clue | キーバインドのヒント表示（which-key相当） |
| mini.cmdline | コマンドライン強化 |
| mini.map | コードミニマップ |

### 編集機能系

| モジュール | 機能 |
|---|---|
| mini.ai | テキストオブジェクト拡張 |
| mini.align | テキスト整列 |
| mini.comment | コメント操作 |
| mini.move | 行・選択範囲の移動 |
| mini.operators | カスタムオペレータ |
| mini.pairs | 自動括弧補完 |
| mini.splitjoin | 引数の分割・結合 |
| mini.surround | 囲み文字操作 |
| mini.trailspace | 末尾の空白を強調表示 |

### ナビゲーション系

| モジュール | 機能 |
|---|---|
| mini.files | ファイルエクスプローラ |
| mini.pick | ファジーファインダー |
| mini.extra | 追加ピッカー |
| mini.jump | 1文字ジャンプ |
| mini.jump2d | ラベルベースの画面内ジャンプ |
| mini.bracketed | ブラケットナビゲーション |
| mini.visits | ファイル訪問履歴 |

### 補完・スニペット系

| モジュール | 機能 |
|---|---|
| mini.completion | 自動補完（LSP対応） |
| mini.snippets | スニペット管理 |

### Git系

| モジュール | 機能 |
|---|---|
| mini.git | Gitコマンド連携 |
| mini.diff | 差分表示・ハンク操作 |

### その他

| モジュール | 機能 |
|---|---|
| mini.basics | 基本設定プリセット（Alt移動等） |
| mini.sessions | セッション管理 |
| mini.misc | ユーティリティ関数集・auto-root設定 |
| mini.bufremove | バッファ削除 |
| mini.hipatterns | パターンのハイライト（TODO/FIXME/カラーコード） |
| mini.indentscope | インデントスコープ表示 |
| mini.keymap | 特殊キーマッピング |
| mini.animate | アニメーション（デフォルト無効） |
| mini.colors | カラースキーム操作（デフォルト無効） |
| mini.cursorword | カーソル下の単語をハイライト（デフォルト無効） |

:::message
`mini.animate`、`mini.colors`、`mini.cursorword`は設定ファイルにコメントアウトされた状態で含まれとる。必要な場合はコメントを外して有効化できる。
:::

## 非miniプラグインの役割

MiniMaxには6つの非miniプラグインが含まれとる：

| プラグイン | 役割 |
|---|---|
| nvim-treesitter/nvim-treesitter | Tree-sitterによる構文解析 |
| nvim-treesitter/nvim-treesitter-textobjects | Tree-sitterを使ったテキストオブジェクト |
| neovim/nvim-lspconfig | 各言語のLSP設定コレクション |
| stevearc/conform.nvim | コードフォーマッタ連携 |
| rafamadriz/friendly-snippets | 各言語の豊富なスニペット集 |
| (mason.nvim) | 言語サーバー管理（コメントアウト・任意） |

## 設定ファイルを自分のリポジトリで管理する

MiniMaxの設定は`~/.config/nvim-minimax/`に生成されとる。これをGitリポジトリにして管理しておくと、他のマシンでも同じ環境を再現できて便利や。

```bash
cd ~/.config/nvim-minimax

# Gitリポジトリとして初期化
git init
git add .
git commit -m "Initial MiniMax configuration"

# GitHubやGitLabにpushしておく
git remote add origin https://github.com/yourname/nvim-config
git push -u origin main
```

:::message
dotfilesとして管理している人は、そこに組み込んでもええで。`~/.config/nvim-minimax/`を`~/.dotfiles/nvim-minimax/`としてシンボリックリンクを貼るのが一般的なやり方や。
:::

## 次のステップ

セットアップはこれで完了や！次の章からは、実際にNeovimを操作する方法を学んでいくで。まずはVimの一番の特徴である「モード」の概念から始めよう。
