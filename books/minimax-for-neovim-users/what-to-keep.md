---
title: "外部プラグインが必要な領域"
---

## mini.nvimが意図的にカバーしないもの

前章で触れたように、mini.nvimのREADMEには明確なno-goalが書かれている。これは欠点ではなく設計判断や。カバーしないことで、カバーすると決めた領域の品質を保てる。

ここでは「mini.nvimで全て置き換えよう」としても**置き換えられない領域**を正直に解説する。

## 必須: nvim-lspconfig

LSP（Language Server Protocol）の設定はmini.nvimの対象外や。

nvim-lspconfigはNeovim公式が管理するリポジトリで、各言語のLSP設定集を提供している。これはmini.nvimが置き換えを目指していないし、目指す理由もない。

```lua
-- lazy.nvimでの設定（従来の方法）
{
  'neovim/nvim-lspconfig',
  config = function()
    local lspconfig = require('lspconfig')
    lspconfig.ts_ls.setup({})
    lspconfig.rust_analyzer.setup({})
    lspconfig.gopls.setup({})
  end,
}
```

ただしNeovim 0.10以降では、`after/lsp/` ディレクトリを使うとよりシンプルに書ける（MiniMaxの0.12設定が採用しているアプローチ）：

```lua
-- plugin/40_plugins.lua（nvim-lspconfigを追加後）
vim.lsp.enable({ 'ts_ls', 'rust_analyzer', 'gopls' })
```

```lua
-- after/lsp/ts_ls.lua（カスタム設定が必要な場合のみ作成）
return {
  on_attach = function(client, bufnr)
    -- バッファローカルなキーマップ等
  end,
}
```

mini.nvimと共存させる場合も、nvim-lspconfigはそのまま使い続ける。

### mason.nvimも選択肢

言語サーバーのインストールを自動化したい場合は`mason.nvim` + `mason-lspconfig.nvim`が便利や。MiniMaxの設定にもコメントアウトされた状態で含まれている。

```lua
{
  'williamboman/mason.nvim',
  config = function()
    require('mason').setup()
  end,
},
{
  'williamboman/mason-lspconfig.nvim',
  config = function()
    require('mason-lspconfig').setup({
      ensure_installed = { 'ts_ls', 'rust_analyzer', 'gopls' },
    })
  end,
}
```

## 必須: nvim-treesitter

Tree-sitterによる構文解析エンジンと、各言語のパーサーを管理するのがnvim-treesitterや。mini.nvimの多くのモジュール（mini.ai、mini.surround等）がTree-sitterの恩恵を受けているが、パーサーのインストール・管理はnvim-treesitterが担う。

```lua
{
  'nvim-treesitter/nvim-treesitter',
  build = ':TSUpdate',
  config = function()
    require('nvim-treesitter.configs').setup({
      ensure_installed = {
        'lua', 'typescript', 'javascript', 'rust', 'go', 'python',
        'json', 'yaml', 'markdown', 'markdown_inline',
      },
      highlight = { enable = true },
      indent = { enable = true },
    })
  end,
}
```

mini.nvimと組み合わせる際の注意点：nvim-treesitter-textobjectsはmini.aiと**相補的に使える**。AST精度が必要なテキストオブジェクト（`@function.outer`等）にはtreesitter-textobjects、その他の高度なテキストオブジェクト操作にはmini.aiを使い分けると最強の組み合わせになる。

## 必須: conform.nvim（またはnull-ls後継）

コードフォーマッタの管理もmini.nvimの対象外や。conform.nvimはMiniMaxの設定にも含まれている。

```lua
{
  'stevearc/conform.nvim',
  config = function()
    require('conform').setup({
      formatters_by_ft = {
        typescript = { 'prettier' },
        javascript = { 'prettier' },
        rust = { 'rustfmt' },
        go = { 'goimports', 'gofmt' },
        python = { 'black', 'isort' },
        lua = { 'stylua' },
      },
      format_on_save = {
        timeout_ms = 500,
        lsp_format = 'fallback',
      },
    })
  end,
}
```

## 状況次第: 補完

前章で触れたように、mini.completionは意図的にミニマルな設計や。

```
mini.completion が十分な場合:
- LSP補完があれば足りる
- コード補完のカスタマイズに興味がない
- 軽量さを最優先にしたい

blink.cmp / nvim-cmp が必要な場合:
- copilot補完等のsource pluginと連携したい
- typo-resistantなfuzzy matchingが欲しい
- 補完のランキングや表示を細かく制御したい
```

`copilot.lua`の自動補完とmini.completionを組み合わせると`<Tab>`が競合しやすい。その場合は明確に blink.cmp か nvim-cmp を選んだ方がストレスが少ない。

## AIコーディングプラグイン

copilot.lua、avante.nvim、codecompanion.nvim、sidekick.nvim——これらはmini.nvimの対象外で、独立して使う。

mini.nvimエコシステムを採用してもAIコーディングプラグインはそのまま使える。

:::message
別のbookでAIコーディングプラグインの選び方・導入方法を詳しく解説しとる（「MiniMaxではじめるNeovim入門」第11章）。
:::

## 最小の完全スタック

mini.nvimで統一したとしても、現実的な完全スタックはこうなる：

```lua
-- 最小の完全Neovim設定（mini.nvim + 必須外部プラグイン）

-- コア: mini.nvim（UI・編集・ナビゲーション・Gitの統一体験）
-- mini.pick, mini.files, mini.ai, mini.surround, mini.diff, mini.git,
-- mini.completion, mini.statusline, mini.tabline, mini.clue, ...

-- 外部（避けられない）
{ 'neovim/nvim-lspconfig' },   -- LSP設定
{ 'nvim-treesitter/nvim-treesitter', build = ':TSUpdate' },  -- 構文解析
{ 'stevearc/conform.nvim' },   -- フォーマッタ

-- オプション（補完を強化したい場合）
{ 'saghen/blink.cmp' },        -- 高機能補完

-- オプション（AIコーディング）
{ 'zbirenbaum/copilot.lua' },  -- or avante, codecompanion, sidekick
```

LazyVimやNvChadのフル設定と比較すると、全体のプラグイン数が大幅に少ない。でも「全機能が揃っている」状態は達成できる。

## 「少ないプラグインが良い」という価値観

ここで一つ問いかけたい。

あなたは今、何個のプラグインを使っているか？ `lazy.nvim`の起動ログや`:Lazy`のプラグイン一覧を見てみてほしい。30個？50個？

多くのプラグインを使うことは悪いことではない。でも、**自分が使っているプラグインを全部把握しているか？** 各プラグインが何をしているか説明できるか？

mini.nvimエコシステムに移行することの副産物として、「何が入っていて、何をしているかを完全に把握している設定」が手に入る。これは、プラグインの更新で設定が壊れた時のデバッグ能力にも直結する。

次の章では、パッケージマネージャ自体の話——`vim.pack`と`mini.deps`の関係について整理するで。
