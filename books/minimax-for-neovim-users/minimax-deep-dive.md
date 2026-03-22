---
title: "MiniMaxを読む・使う"
---

## ディストリビューションとの違いを明確にする

MiniMaxを理解するには、まず「ディストリビューションではない」ということを体感する必要がある。

| | LazyVim / NvChad | MiniMax |
|---|---|---|
| 更新 | 上流の変更が自動反映 | 生成後は自分で管理 |
| 目的 | 「使う」 | 「生成して読んで学ぶ」 |
| 設定の所有者 | 上流リポジトリ | あなた |
| カスタマイズ | プロジェクトの規約に従う | 生成後は自由 |
| 学習効果 | 設定を読まなくても使える | 読むことが前提 |

MiniMaxと近い思想を持つのは[kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim)や。どちらも「設定を自分で理解・管理してほしい」という哲学がある。MiniMaxが異なるのは、**mini.nvimで統一されたエコシステムを採用している**点と、**Neovimバージョン別に最適化された設定**が用意されている点や。

## NVIM_APPNAMEで安全に試す

既存の設定を壊さずにMiniMaxを試せる。`NVIM_APPNAME`環境変数でNeovimの設定ディレクトリを切り替えられるからや。

```bash
# MiniMaxをクローン（どこでもOK）
git clone --filter=blob:none https://github.com/nvim-mini/MiniMax ./MiniMax

# セットアップスクリプトを実行
# → ~/.config/nvim-minimax/ に設定ファイルが生成される
NVIM_APPNAME=nvim-minimax nvim -l ./MiniMax/setup.lua

# MiniMaxを使ったNeovimを起動
NVIM_APPNAME=nvim-minimax nvim
```

エイリアスを設定しておくと便利や：

```bash
alias nvim-mm='NVIM_APPNAME=nvim-minimax nvim'
```

これで `nvim-mm` コマンドでMiniMax版Neovimが起動する。普段の `nvim` は一切変わらない。

## 生成されたファイルを読む

セットアップで生成された設定ファイルを実際に開いてみてほしい。

```bash
ls ~/.config/nvim-minimax/plugin/
# 10_options.lua  20_keymaps.lua  30_mini.lua  40_plugins.lua
```

### 30_mini.lua が最も価値がある

`plugin/30_mini.lua` は全mini.nvimモジュールの設定が書かれた、このbookで一番重要なファイルや。

```bash
wc -l ~/.config/nvim-minimax/plugin/30_mini.lua
# 400〜600行程度
```

**このファイルのコメントを読むだけで、mini.nvimの使い方が分かる**。たとえばmini.clueの設定を抜粋すると：

```lua
-- plugin/30_mini.lua（コメントを含む実際の設定例）
--
-- mini.clue provides "key clues": hint about what next key would do.
-- Clues appear in a floating window after user stopped typing for
-- `config.window.delay` milliseconds (500 by default).
require('mini.clue').setup({
  triggers = {
    -- Leader triggers
    { mode = 'n', keys = '<Leader>' },
    { mode = 'x', keys = '<Leader>' },

    -- Built-in completion
    { mode = 'i', keys = '<C-x>' },

    -- `g` key
    { mode = 'n', keys = 'g' },
    { mode = 'x', keys = 'g' },

    -- Marks
    { mode = 'n', keys = "'" },
    { mode = 'n', keys = '`' },

    -- Registers
    { mode = 'n', keys = '"' },
    { mode = 'i', keys = '<C-r>' },

    -- Window commands
    { mode = 'n', keys = '<C-w>' },

    -- `z` key
    { mode = 'n', keys = 'z' },
    { mode = 'x', keys = 'z' },
  },

  clues = {
    -- Enhance this by adding descriptions for <Leader> mapping groups
    MiniClue.gen_clues.builtin_completion(),
    MiniClue.gen_clues.g(),
    MiniClue.gen_clues.marks(),
    MiniClue.gen_clues.registers(),
    MiniClue.gen_clues.windows(),
    MiniClue.gen_clues.z(),
  },
})
```

設定の意図がコメントで明確に説明されとる。これを読むと「なぜこの設定になっているのか」が理解できる。

### バージョン別設定を比較する

MiniMaxはNeovim 0.10/0.11/0.12それぞれの設定を提供している。バージョン間の差分を見ることで、Neovimがどう進化してきたかが分かる。

```bash
# GitHubでバージョン間の差分を確認
# https://github.com/nvim-mini/MiniMax/compare/nvim-0.11...nvim-0.12
```

0.11→0.12の最大の変化は`mini.deps`から`vim.pack`への移行や。これはNeovim本体にパッケージマネージャが組み込まれたことを意味する（詳細は後の章で）。

## MiniMaxブログを活用する

[nvim-mini.org/blog](https://nvim-mini.org/blog/) には、mini.nvimとMiniMaxの開発状況が詳しく書かれている。

主要な記事：

- `2025-10-13-announce-minimax.html`: MiniMaxのアナウンス、設計思想の説明
- `2026-02-15-update-minimax-configs-012-010.html`: 0.12対応とvim.packへの移行
- `2024-03-28-announce-mini-diff.html`: mini.diffのアナウンス

これらを読むと、「MiniMaxがなぜこの設計になっているのか」「mini.nvimエコシステムがどこへ向かっているのか」が見えてくる。

## kickstart.nvimとの比較

「設定を生成して自分で管理する」という思想を持つもう一つの有名プロジェクトがkickstart.nvimや。

```
kickstart.nvim:
- 単一ファイル (init.lua)
- lazy.nvimを使用
- LSP、telescope、nvim-cmp等をゼロから設定する教育目的
- プラグイン選択に特定のエコシステム縛りなし

MiniMax:
- 複数ファイル (plugin/10〜40)
- vim.pack（0.12）/ mini.deps（0.10/0.11）を使用
- mini.nvimエコシステムで統一
- バージョン別設定が用意されている
```

どちらが良いかではなく、**kickstart.nvimは「一から学ぶ」人向け、MiniMaxは「mini.nvimエコシステムで統一した設定の出発点が欲しい」人向け**と理解するといい。

## MiniMaxを設定の「参考書」として使う

MiniMaxを完全に採用しなくても、設定ファイルを「リファレンス実装」として使える。

たとえば、「mini.diffの設定例が知りたい」と思ったら：

```bash
grep -A 20 "mini.diff" ~/.config/nvim-minimax/plugin/30_mini.lua
```

コメント付きの実用的な設定例がすぐ見つかる。ドキュメントを読むより、動いている設定を読む方が理解しやすいことは多い。

## まとめ

MiniMaxの価値は2つある：

1. **mini.nvimエコシステムで動く完全な設定の出発点** として使う
2. **mini.nvimの各モジュールをどう設定するかの参考書** として使う

どちらの使い方でも、`NVIM_APPNAME`で既存設定を壊さずに試せるのが大きな利点や。

次の章では、実際に既存設定からmini.nvimへどう移行するかの戦略を考えよう。
