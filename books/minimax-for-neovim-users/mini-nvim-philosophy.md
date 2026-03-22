---
title: "mini.nvimの設計哲学"
---

## なぜ「一人で作る」のか

mini.nvimを最初に知ったとき、「40以上のプラグインを一人で？」と思う人は多い。でもこれはバグではなくフィーチャーや。

echasnovskiが一人で全モジュールを開発・管理することで、**設計上の一貫性が保証される**。各モジュールの設定方法、ドキュメントの書き方、キーマップの命名規則、アップデートの思想——全てが同じ人間の判断軸で統一されとる。

複数の著者が作ったプラグインを組み合わせる場合と比較してみよう：

```lua
-- 典型的なlazy.nvim設定での「バラバラ感」
-- telescope: opts.defaults, opts.pickers, ...
-- nvim-cmp: sources = { {name='nvim_lsp'}, ... }
-- gitsigns: on_attach = function(bufnr) ... end
-- lualine: sections = { lualine_a = {'mode'}, ... }
-- nvim-surround: keymaps = { insert = '<C-g>s' }
-- which-key: { '<leader>f', group = 'file' }

-- mini.nvimでの統一感
require('mini.pick').setup({})
require('mini.completion').setup({})
require('mini.diff').setup({})
require('mini.statusline').setup({})
require('mini.surround').setup({})
require('mini.clue').setup({})
```

全部 `require('mini.xxx').setup({})` で始まる。設定テーブルのキー名の命名規則も統一されとる。`opts.mappings`や`opts.config`のような不規則さがない。

一人で作るということは、「このプラグインはこのやり方、あっちはそのやり方」という分裂が起きないということや。

## 意図的なスコープの限定

mini.nvimのREADMEにはこう書いてある：

> Including functionality which needs several filetype/language specific implementations is an explicit no-goal.

言語固有の実装を含めることは**明確に目標としていない**。

これは重要な設計判断や。LSP設定、言語サーバーのインストール、フォーマッタの管理——こういった領域にmini.nvimは踏み込まない。踏み込まない理由は品質と保守性のためや。

「全部やろうとしない」という割り切りが、**やると決めた機能の品質を高める**。

### feasibleな実装だけを含む

mini.nvimの各モジュールは「実現可能で、保守可能で、一貫したものだけ」という基準で作られとる。中途半端な実装を入れるくらいなら、そのスコープは別のプラグインに任せる。

この潔さが、mini.nvimモジュールの安定性につながっとる。機能が少ない分、バグも少ない。

## 各モジュールの独立性

mini.nvimの各モジュールは**互いに依存しない**。`mini.pick`を使うために`mini.files`が必要、ということは起きない。

```lua
-- mini.surroundだけ使う（他は何もいらない）
require('mini.surround').setup({
  mappings = {
    add = 'gsa',
    delete = 'gsd',
    replace = 'gsr',
  },
})
```

これだけで動く。他のmini.nvimモジュールは一切不要や。

この独立性が、「段階的に試す」ことを可能にする。今の設定を全部捨てずに、気になるモジュールだけ追加して試せる。

## Tree-sitter優先設計

mini.nvimの多くのモジュールはTree-sitterに対応した設計になっとる：

- **mini.ai**: Tree-sitterを使ったより精密なテキストオブジェクト
- **mini.surround**: Tree-sitterによるHTMLタグの認識
- **mini.diff**: Tree-sitterを使ったsyntax-aware diff
- **mini.indentscope**: インデントスコープの視覚化

Neovimの組み込みTree-sitter対応が成熟したことで、これらの恩恵をフルに受けられる。

## 「40+モジュールを一人で」の持続可能性

「一人で作ると、いつか開発が止まるのでは？」という懸念は理解できる。ただし現状を見ると：

- **2025年8月**: 専用GitHubオーガニゼーション（nvim-mini）に移行
- **2025年10月**: 専用サイト（nvim-mini.org）立ち上げ＆MiniMaxリリース
- **2026年2月**: Neovim 0.12対応の設定を先行公開
- **継続的な更新**: 2024〜2026年にかけて新モジュールのリリースが続いている

組織化とインフラ整備が進んでいる。単なる個人プロジェクトから、コミュニティに支えられたエコシステムへと成長しつつある。

また、**LazyVimがmini.aiをデフォルトとして採用**していることは、外部のプロジェクトがmini.nvimの品質を認めているという証拠や。一度外して「失敗だった」として戻したという経緯も、mini.aiがLazyVimユーザーに受け入れられている事実を示しとる。

## mini.nvimが伝えたい思想

echasnovskiが作るものの根底には「**Neovimを使い倒すためのツールを、Neovimの思想に沿って作る**」という哲学があると感じる。

Vimは「シンプルなプリミティブを組み合わせる」文化がある。オペレータ+モーション、テキストオブジェクト、ドットリピート——これらはシンプルな部品が強力に組み合わさる設計や。

mini.nvimはその思想をプラグイン設計にも持ち込んでいる。`mini.operators`がVimのオペレータの概念を拡張し、`mini.ai`がテキストオブジェクトをシームレスに拡張する。Vimの哲学を壊さずに拡張するアプローチや。

次の章では、具体的なプラグインとの比較を通じて、この哲学が実際の操作体験にどう現れるかを見ていくで。
