---
title: "よく使うプラグインとの比較"
---

## 比較の前置き

この章では、よく使われているプラグインとmini.nvimモジュールを正直に比較する。「mini.nvimの方が優れている」という結論に持っていくつもりはない。

大事なのは**何を得て、何を失うか**を理解することや。

## ピッカー: telescope.nvim → mini.pick + mini.extra

### telescope.nvim

telescopeは説明不要の定番ピッカーや。巨大なエコシステム、豊富な組み込みピッカー、fzf-native extensionによる高速マッチング、無数のコミュニティ製extension——機能的には最強クラス。

```lua
-- telescopeの豊富な設定例
require('telescope').setup({
  defaults = {
    file_ignore_patterns = { 'node_modules', '.git/' },
    layout_strategy = 'horizontal',
    sorting_strategy = 'ascending',
  },
  extensions = {
    fzf = {
      fuzzy = true,
      override_generic_sorter = true,
    },
  },
})
```

### mini.pick + mini.extra

mini.pickはシンプルな単一ウィンドウのピッカー。mini.extraが追加のピッカー（LSP、git log等）を提供する。

```lua
-- mini.pickはこれだけで動く
require('mini.pick').setup({})

-- 主要なピッカー
vim.keymap.set('n', '<leader>ff', MiniPick.builtin.files)
vim.keymap.set('n', '<leader>fg', MiniPick.builtin.grep_live)
vim.keymap.set('n', '<leader>fb', MiniPick.builtin.buffers)
vim.keymap.set('n', '<leader>fh', MiniPick.builtin.help)
```

**mini.pickの強み**:
- 依存なし（外部バイナリ不要）
- シンプルで予測可能なUI
- Lua APIが整理されていてカスタムピッカーが作りやすい

**telescopeの強み**:
- extensionsが圧倒的に豊富
- fzf統合による高速マッチング
- コミュニティのサポートが厚い

**判断軸**: telescope-extensionsを多用しているなら乗り換えメリットは薄い。シンプルな操作（ファイル検索・grep・バッファ）だけで十分なら、mini.pickは快適な選択肢や。

---

## 補完: nvim-cmp / blink.cmp → mini.completion

### 2025-2026年の補完事情

2025年後半から、LazyVimが`nvim-cmp`から`blink.cmp`にデフォルトを切り替えた。blink.cmpは：

- 起動オーバーヘッド 0.5-4ms
- typo-resistantなfuzzy matching
- LSP、バッファ、パス等の組み込みsource

nvim-cmpは依然として広く使われているが、新規セットアップではblink.cmpを選ぶケースが増えとる。

### mini.completion

mini.completionはNeovimの組み込み補完機能に薄いラッパーを追加したもの。意図的にミニマルな設計や。

```lua
require('mini.completion').setup({
  lsp_completion = {
    source_func = 'omnifunc',
    auto_setup = false,
  },
  -- 基本的にはこれだけ
})
```

**mini.completionの強み**:
- 設定がほぼ不要、すぐ動く
- LSP補完が自然に効く
- 軽量

**blink.cmp / nvim-cmpの強み**:
- source pluginエコシステムが豊富（copilot-cmp、crates.nvim等との連携）
- fuzzyマッチングが高精度
- 補完UIの細かいカスタマイズが可能

**判断軸**: mini.completionはスターターとして優秀。でも、copilot補完や特殊なsourceを使いたい、補完のランキングや表示を細かく制御したい、という場合はblink.cmpが現実的な選択や。

---

## ファイルツリー: neo-tree / nvim-tree → mini.files

### mini.filesの独特なUX

mini.filesは他のファイルツリーと根本的にUIが違う。**ファイルシステムをバッファとして操作する**設計や。

```
左ペイン     中ペイン      右ペイン（プレビュー）
src/       ├ components/ ├ Header.tsx
  ├ comp/  │ ├ Header.tsx │   import React from 'react'
  └ pages/ │ └ Footer.tsx │   ...
```

ファイルのリネームが `C`（リネームモードに入った後 `=` で同期）、削除が `dd`、移動が `yy` / `p` でできる。Vimの操作体系でファイル管理ができる。

```lua
require('mini.files').setup({
  windows = {
    preview = true,
    width_focus = 30,
    width_preview = 50,
  },
})
```

**mini.filesの強み**:
- Vimの操作体系でファイル操作できる
- フローティングウィンドウで邪魔にならない
- テキスト操作と同じ感覚でファイル整理

**neo-tree / nvim-treeの強み**:
- 伝統的なサイドバー型UIに慣れているなら直感的
- copy/pasteのUXが分かりやすい
- アイコン、カラーリング等の視覚的な豊富さ

**判断軸**: 「Vimのモーションでファイル操作したい」という人には刺さる。伝統的なサイドバーが好きな人には合わないかもしれない。一度試してみる価値はある。

---

## Git差分: gitsigns.nvim → mini.diff

### mini.diffの差別化ポイント

mini.diffはgitsigns.nvimとは少し違う立ち位置や。

```lua
require('mini.diff').setup({
  view = {
    style = 'sign',     -- または 'number'
    signs = { add = '▎', change = '▎', delete = '▏' },
  },
})
```

**mini.diffが優れている点**:

1. **histogram diff + linematch**: デフォルトのアルゴリズムが違う。変更箇所の視認性が高い。
2. **Gitに縛られない**: referenceをGitの`HEAD`以外（ステージングエリア、任意のブランチ等）に設定できる。
3. **mini.gitとの統合**: `mini.git`と合わせて使うとNeovim内でのGit操作がシームレスになる。

**gitsigns.nvimが優れている点**:

- git blame（行ごとのコミット情報表示）
- hunk staging（ハンク単位のステージ）が充実
- 長い歴史と大きなコミュニティ

**判断軸**: `git blame`をVim内で頻繁に使うならgitsignsの方が快適。差分の視覚的な確認と基本的なhunk操作だけなら、mini.diffは十分以上や。

---

## テキストオブジェクト: nvim-treesitter-textobjects → mini.ai

これは個人的に最もmini.nvimが光る領域の一つや。

### mini.aiの強み

```lua
require('mini.ai').setup({
  n_lines = 500,  -- 検索範囲を広げる
})

-- 使用例
-- cia: 引数を変更（Change Inner Argument）
-- dan: 次の括弧ごと削除（Delete Around Next）
-- 2ciq: 2番目のクォート内を変更
```

1. **v:count対応**: `2ciw` で2番目の単語を変更、`3di"` で3番目のクォート内を削除
2. **consecutive application**: 同じオブジェクトを連続して操作できる
3. **複数の検索方法**: next（次）、previous（前）、nearest（最近傍）
4. **dot-repeat対応**: `.` で繰り返せる

### nvim-treesitter-textobjectsとの比較

```lua
-- treesitter-textobjectsの設定例
require('nvim-treesitter.configs').setup({
  textobjects = {
    select = {
      enable = true,
      lookahead = true,
      keymaps = {
        ['af'] = '@function.outer',
        ['if'] = '@function.inner',
      },
    },
  },
})
```

- **treesitter-textobjectsが優れている点**: Tree-sitterのAST（構文木）に完全準拠した精度
- **mini.aiが優れている点**: v:count、dot-repeat、連続適用という「Vimらしい」操作感

実はこの2つは**相補的**で、両方使っているユーザーも多い。mini.aiで基本的なテキストオブジェクトを、treesitter-textobjectsで`@function.outer`のようなAST精度が必要なものを使い分ける。

---

## キーヒント: which-key.nvim → mini.clue

### 設計思想の違い

which-keyはキーマップの定義と表示を一体化してる。mini.clueは**表示だけを担当**する。

```lua
-- which-keyでのキーマップ定義（表示と定義が混在）
require('which-key').add({
  { '<leader>f', group = 'file' },
  { '<leader>ff', '<cmd>Telescope find_files<cr>', desc = 'Find File' },
})

-- mini.clueでは、キーマップは別で定義してヒントだけ設定
vim.keymap.set('n', '<leader>ff', '<cmd>Pick files<cr>', { desc = 'Find files' })

require('mini.clue').setup({
  clues = {
    { mode = 'n', keys = '<Leader>f', desc = 'Find...' },
    MiniClue.gen_clues.builtin_completion(),
    MiniClue.gen_clues.g(),
    MiniClue.gen_clues.z(),
  },
  triggers = {
    { mode = 'n', keys = '<Leader>' },
    { mode = 'n', keys = 'g' },
  },
})
```

**mini.clueの強み**: 関心の分離。キーマップ定義は`vim.keymap.set`で、ヒント表示はmini.clueで、という責務の明確化。

**which-keyの強み**: 既存キーマップの`desc`を自動で読み取るので追加設定が少ない。`v3`のような現在のバージョンのUIも充実している。

---

## lualine → mini.statusline

差は一言で言うと「シンプルさ vs 表現力」や。

```lua
-- mini.statusline: シンプル
require('mini.statusline').setup({
  use_icons = true,
})

-- lualine: 細かく制御できる
require('lualine').setup({
  sections = {
    lualine_a = {'mode'},
    lualine_b = {'branch', 'diff', 'diagnostics'},
    lualine_c = {'filename'},
    lualine_x = {'encoding', 'fileformat', 'filetype'},
  },
  options = {
    theme = 'tokyonight',
    component_separators = '|',
  },
})
```

mini.statuslineはmini.git、mini.diff、mini.iconsとよく連携する。lualineは多くのthemeや細かいレイアウト設定が可能。ステータスラインにこだわりがなければmini.statuslineで十分やけど、凝りたい人はlualineで良い。

---

## 比較表まとめ

| 機能 | mini.nvim | 対応プラグイン | mini.nvimを選ぶ理由 | 対応プラグインを選ぶ理由 |
|---|---|---|---|---|
| ピッカー | mini.pick | telescope | シンプル・依存なし | extensions豊富 |
| 補完 | mini.completion | blink.cmp | 設定不要 | 高機能・高精度 |
| ファイルツリー | mini.files | neo-tree | Vimの操作感 | 視覚的UIが豊富 |
| Git差分 | mini.diff | gitsigns | histogram diff | blame・staging |
| テキストオブジェクト | mini.ai | ts-textobjects | v:count・dot-repeat | AST精度 |
| キーヒント | mini.clue | which-key | 関心の分離 | 自動desc読み取り |
| ステータスライン | mini.statusline | lualine | エコシステム統合 | カスタマイズ性 |

**共通して言えること**: mini.nvimを選ぶ理由は「シンプル・統一・依存なし」。対応プラグインを選ぶ理由は「機能・拡張性・コミュニティ」。どちらが正解かではなく、自分が何を重視するかの問題や。

次の章では、MiniMaxを実際に動かして設定ファイルを読んでみよう。
