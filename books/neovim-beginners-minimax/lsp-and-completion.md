---
title: "LSPとコード補完で快適なコーディング"
---

## LSPとは

**LSP**（Language Server Protocol）は、マイクロソフトが2016年に策定したプロトコルや。エディタ（クライアント）と言語解析サーバー（サーバー）の通信規格を統一したもので、これにより各エディタが全プログラミング言語に対応するコードを書かずに済むようになった。

```
┌─────────────┐         ┌─────────────────────┐
│   Neovim    │ ←─LSP─→ │  Language Server    │
│  (クライアント) │         │  (TypeScript, Rust等) │
└─────────────┘         └─────────────────────┘
```

LSPを通じて以下の機能が使えるようになる：

- **コード補完**: 入力中に候補を表示
- **定義へジャンプ**: 関数や変数の定義元に移動
- **参照の検索**: どこから呼ばれているかを調べる
- **ホバー情報**: 変数や関数の型・ドキュメントを表示
- **エラー診断**: リアルタイムでエラーや警告を表示
- **リファクタリング**: リネームや自動修正

## 言語サーバーのインストール

MiniMaxには `nvim-lspconfig` が含まれていて、各言語のLSP設定が簡単に書けるようになっとる。ただし言語サーバー自体はNeovimとは別にインストールが必要や。

### TypeScript / JavaScript

```bash
npm install -g typescript-language-server typescript
# または
pnpm add -g typescript-language-server typescript
```

### Python

```bash
pip install pyright
# または軽量版
pip install python-lsp-server
```

### Go

```bash
go install golang.org/x/tools/gopls@latest
```

### Rust

```bash
# rustupをインストール済みの場合、rust-analyzerも一緒に入る
rustup component add rust-analyzer

# または直接インストール
brew install rust-analyzer  # macOS
```

### Lua

```bash
# macOS
brew install lua-language-server

# Linux（バイナリをダウンロード）
# https://github.com/LuaLS/lua-language-server/releases
```

### Ruby

```bash
gem install solargraph
```

### 言語サーバーを有効化する

MiniMaxでは、Neovim組み込みの仕組みを使って言語サーバーを設定する。2ステップや：

**ステップ1**: `after/lsp/` ディレクトリにサーバー設定ファイルを作る

```lua
-- after/lsp/ts_ls.lua（TypeScript用）
return {}  -- デフォルト設定でOK。カスタム設定が必要なら追記する
```

```lua
-- after/lsp/pyright.lua（Python用）
return {}
```

```lua
-- after/lsp/gopls.lua（Go用）
return {}
```

```lua
-- after/lsp/rust_analyzer.lua（Rust用）
return {}
```

`after/lsp/lua_ls.lua` はMiniMaxに最初から含まれとる（Lua用の細かい設定が書いてある）。

**ステップ2**: `plugin/40_plugins.lua` でサーバーを有効化する

```lua
-- plugin/40_plugins.lua（nvim-lspconfigの設定部分に追記）
vim.lsp.enable({
  'ts_ls',      -- TypeScript
  'pyright',    -- Python
  'gopls',      -- Go
  'rust_analyzer', -- Rust
  'lua_ls',     -- Lua
})
```

:::message
`after/lsp/` はNeovim 0.10以降に追加された組み込み機能や。ファイルを置くだけで `vim.lsp.enable()` 時に自動的に読み込まれる。`nvim-lspconfig` が提供する各言語のデフォルト設定の上に、自分のカスタム設定を重ねることができる仕組みや。
:::

:::message
**mason.nvimを使う方法**

`plugin/40_plugins.lua`にはmason.nvimの設定がコメントアウトで含まれとる。mason.nvimを有効にすると、Neovim内から言語サーバーをインストール・管理できるようになる。

コメントを外して有効化すれば`:MasonInstall typescript-language-server`のような操作ができるようになる。言語サーバーを複数管理する予定があればmasonが便利や。
:::

## LSPの基本操作

言語サーバーが正常に起動していれば、以下のキーマップが使える（MiniMaxのデフォルト設定）：

### 定義・参照の操作

```vim
gd     " 定義へジャンプ（Go to Definition）
gD     " 宣言へジャンプ（Go to Declaration）
gr     " 参照の一覧（Go to References）
gI     " 実装へジャンプ（Go to Implementation）
gy     " 型定義へジャンプ
```

`gd`でジャンプした後、`Ctrl-o`で元の場所に戻れる。

### 情報の表示

```vim
K      " ホバー情報を表示（カーソル下のシンボルの型・ドキュメント）
<C-k>  " シグネチャヘルプ（関数の引数情報）
```

### 診断（エラー・警告）の操作

```vim
[d            " 前の診断へ移動
]d            " 次の診断へ移動
<Leader>ld    " 診断の詳細をフローティングウィンドウで表示
<Leader>fd    " ワークスペース全体の診断一覧（mini.pick経由）
<Leader>fD    " 現在のバッファの診断一覧（mini.pick経由）
```

エラーは画面上に直接表示される（`mini.diff`との連携でガターにも表示）。

### コード操作

```vim
<Leader>lr    " リネーム（Rename）
<Leader>la    " コードアクション（Code Action）
<Leader>li    " 実装へジャンプ（Implementation）
<Leader>ls    " ソース定義へジャンプ（Source definition）
<Leader>lt    " 型定義へジャンプ（Type definition）
<Leader>lR    " 参照一覧（References）
```

**コードアクションの例：**

```typescript
// import { useState } from 'react'; を入れ忘れた場合
// useStateにカーソルを置いて <Leader>ca
// → "Add import 'useState' from 'react'" というアクションが提案される
```

### LSPの状態確認

```vim
:LspInfo    " 現在のLSP接続状況を確認
:LspStart   " LSPを手動で起動
:LspStop    " LSPを停止
```

## mini.completionによる自動補完

MiniMaxには `mini.completion` が含まれとる。LSPとシームレスに連携して、コード補完を提供してくれる。

### 補完の操作

コーディング中に文字を入力すると、自動的に補完候補のポップアップが表示される：

```vim
Ctrl-n    " 次の候補を選択
Ctrl-p    " 前の候補を選択
Enter     " 候補を確定
Ctrl-e    " 補完を閉じる
Ctrl-Space " 手動で補完を呼び出す
```

### シグネチャヘルプ

関数の引数を入力中に`(`を押すと、シグネチャヘルプ（引数の型情報）が表示される：

```typescript
// getUserById( を入力すると
// getUserById(id: string, options?: GetUserOptions): Promise<User>
// のような情報がポップアップで表示される
```

## mini.snippetsとfriendly-snippets

MiniMaxには `mini.snippets` と `friendly-snippets`（よく使うスニペット集）が含まれとる。

### スニペットの使い方

```vim
" JavaScriptファイルで
log<C-j>   " console.log() が展開される
fun<C-j>   " function が展開される
" プレフィックスが思い出せない場合: <C-j>だけ押すとピッカーが表示される
```

展開後は`<Tab>`でプレースホルダー間を移動できる：

```javascript
// fun<C-j> を展開すると
function functionName(params) {
//       ^-- ここが選択状態

}
// <Tab>を押すとparamsへ、さらに<Tab>でbody部分へ移動
```

### カスタムスニペットの作成

`~/.config/nvim-minimax/snippets/global.json` にJSON形式でスニペットを追加できる：

```json
{
  "console.log": {
    "prefix": "cl",
    "body": "console.log($1);",
    "description": "console.log"
  },
  "todo comment": {
    "prefix": "todo",
    "body": "// TODO: $1",
    "description": "TODO comment"
  }
}
```

言語ごとのスニペットは `after/snippets/typescript.json` のように言語名のファイルを作る。

## conform.nvimでコードをフォーマットする

`conform.nvim` はコードフォーマッタの統合管理ツールや。MiniMaxに含まれとる。

### フォーマッタのインストール

```bash
# JavaScript/TypeScript
npm install -g prettier

# Python
pip install black ruff

# Go（goinstallで入る）
go install golang.org/x/tools/cmd/goimports@latest

# Rust（rustup経由）
rustup component add rustfmt
```

### フォーマット実行

```vim
<Leader>lf  " 現在のバッファをフォーマット
```

または設定で保存時に自動フォーマットを有効にする：

```lua
-- plugin/40_plugins.lua（conform.nvimの設定に追記）
require('conform').setup({
  format_on_save = {
    timeout_ms = 500,
    lsp_format = 'fallback',
  },
  formatters_by_ft = {
    javascript = { 'prettier' },
    typescript = { 'prettier' },
    python = { 'black' },
    go = { 'goimports' },
    rust = { 'rustfmt' },
  },
})
```

## Tree-sitterの設定

MiniMaxには `nvim-treesitter` が含まれとる。Tree-sitterは高精度な構文解析エンジンで、シンタックスハイライトやテキストオブジェクトの精度が大幅に向上する。

### パーサーのインストール

```vim
:TSInstall typescript
:TSInstall python
:TSInstall go
:TSInstall rust
:TSInstall lua
:TSInstall json
:TSInstall yaml
:TSInstall markdown

" または全パーサーを更新
:TSUpdate all
```

### mini.hipatternsによるハイライト

`mini.hipatterns` は特定のパターンを強調表示してくれる：

- `TODO:` → 青色でハイライト
- `FIXME:` → 赤色でハイライト
- `HACK:` → 黄色でハイライト
- `#ff0000` → 実際の色でハイライト（カラーコード）

## 実践：TypeScriptプロジェクトでのLSP体験

実際の開発フローを見てみよう：

```bash
# プロジェクトを開く
cd my-ts-project
NVIM_APPNAME=nvim-minimax nvim src/app.ts
```

```typescript
// src/app.ts でコードを書く

// 1. import を書き始めると補完が効く
import { use  // ← Ctrl-Space で useStateなどが補完される

// 2. 関数を呼ぶ際にシグネチャが表示される
const user = getUserById(  // ← 引数の型情報が表示される

// 3. エラーがあれば即座に赤くなる
const x: number = "hello"  // ← 型エラーが表示される

// 4. gd で定義にジャンプ
getUserById  // ← gd で関数の定義元へ

// 5. <Leader>lr でリネーム
getUserById  // ← <Leader>lr で全ファイルのgetByIdを一括リネーム
```

## まとめ

LSP環境が整うと、Neovimでの開発体験がIDEに近づく。ポイントは：

1. 言語サーバーをシステムにインストールする
2. `after/lsp/` にサーバー設定ファイルを作り、`vim.lsp.enable()` で有効化する
3. `gd`, `K`, `<Leader>lr`, `<Leader>la` などのキーマップを覚える

次の章では、Git連携について学んでいくで！
