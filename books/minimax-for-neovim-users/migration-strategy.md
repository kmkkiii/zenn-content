---
title: "段階的移行のアプローチ"
---

## 全部一気に捨てなくていい

「mini.nvimに移行する」と聞くと、「今の設定を全部消して作り直す」というイメージを持つかもしれない。でもそうしなくていい。

mini.nvimの各モジュールは独立しているので、**今の設定に追加する形で少しずつ試せる**。逆方向も同じ——気に入らなければ外すだけで良い。

ここでは現実的な段階的移行のアプローチを紹介する。

## ステップ0: NVIM_APPNAMEで並行運用

まず最も安全なアプローチとして、MiniMaxを全く別の環境として動かしてみる。前章で説明した方法や：

```bash
git clone --filter=blob:none https://github.com/nvim-mini/MiniMax ./MiniMax
NVIM_APPNAME=nvim-minimax nvim -l ./MiniMax/setup.lua
alias nvim-mm='NVIM_APPNAME=nvim-minimax nvim'
```

普段の作業は `nvim` で続ける。興味を持ったときに `nvim-mm` で試す。この段階では何も失わない。

## ステップ1: 今の設定に mini.ai / mini.surround を追加

最初に試すとすれば、**単体で完結する編集系モジュール**がおすすめや。今のプラグイン構成と競合しない。

### mini.aiを既存設定に追加する

```lua
-- lazy.nvimの設定に追加する場合
{
  'echasnovski/mini.ai',
  version = false,
  config = function()
    require('mini.ai').setup({
      n_lines = 500,
    })
  end,
}
```

追加するだけで `cia`（引数を変更）、`2di"`（2番目のダブルクォート内を削除）のような操作が使えるようになる。nvim-treesitter-textobjectsと**共存できる**ので今の設定を壊す心配がない。

### mini.surroundを追加する

```lua
{
  'echasnovski/mini.surround',
  version = false,
  config = function()
    require('mini.surround').setup({
      mappings = {
        add = 'gsa',            -- 追加 (nvim-surroundと被らないようにgsaに)
        delete = 'gsd',         -- 削除
        find = 'gsf',           -- 検索（前方）
        find_left = 'gsF',      -- 検索（後方）
        highlight = 'gsh',      -- ハイライト
        replace = 'gsr',        -- 置換
        update_n_lines = 'gsn', -- 検索行数を更新
      },
    })
  end,
}
```

`sa`ではなく`gsa`にしているのはvim-surroundやnvim-surroundとの競合を避けるため。既存のsurroundプラグインを使っているなら、まず`gsa`系でmini.surroundを試して、気に入ったら切り替えればいい。

### mini.commentを追加する

`gcc`でコメントトグルできる。nvim-commentやComment.nvimを使っているなら、それらを外してmini.commentに乗り換えやすい。

```lua
{
  'echasnovski/mini.comment',
  version = false,
  config = function()
    require('mini.comment').setup()
  end,
}
```

## ステップ2: mini.files / mini.pickを試す

ファイル操作系を試す段階や。ここからは既存プラグインとの**切り替え**が必要になる（両方同時に使うと混乱するので）。

### mini.filesをneo-tree/nvim-treeと切り替える

まず無効化せずに、別のキーマップでmini.filesを試してみる：

```lua
-- 既存のファイルツリーはそのまま残し
-- mini.filesを別のキーで試す
{
  'echasnovski/mini.files',
  version = false,
  config = function()
    require('mini.files').setup({
      windows = {
        preview = true,
        width_focus = 30,
        width_preview = 50,
      },
    })
    -- 既存の<leader>eとは別に<leader>mfで試す
    vim.keymap.set('n', '<leader>mf', MiniFiles.open, { desc = 'mini.files' })
  end,
}
```

1〜2週間使ってみて、mini.filesのUXが合うかどうか判断する。合うなら既存のファイルツリーを外す。

### mini.pickをtelescopeと切り替える

```lua
{
  'echasnovski/mini.pick',
  'echasnovski/mini.extra',
  config = function()
    require('mini.pick').setup()
    require('mini.extra').setup()

    -- 既存のtelescope keymapと別のprefixで試す
    local pick = MiniPick.builtin
    vim.keymap.set('n', '<leader>mp', pick.files, { desc = 'mini.pick files' })
    vim.keymap.set('n', '<leader>mg', pick.grep_live, { desc = 'mini.pick grep' })
  end,
}
```

telescopeに慣れているなら最初は違和感があるかもしれないけど、1〜2週間使い続けると慣れてくる。

## ステップ3: mini.diff + mini.gitに切り替える

gitsigns.nvimをmini.diff + mini.gitに置き換えるステップや。

```lua
-- gitsigns.nvimをlazy.nvimから外す
-- mini.diff と mini.git を追加する

{
  'echasnovski/mini.diff',
  version = false,
  config = function()
    require('mini.diff').setup({
      view = {
        style = 'sign',
        signs = { add = '▎', change = '▎', delete = '▏' },
      },
    })
    -- ハンク操作のキーマップ
    vim.keymap.set('n', '<leader>gs', function() MiniDiff.toggle_overlay() end, { desc = 'Toggle diff overlay' })
  end,
},
{
  'echasnovski/mini.git',
  version = false,
  config = function()
    require('mini.git').setup()
  end,
}
```

:::message alert
**注意**: gitsigns.nvimの`git blame`機能はmini.gitでは提供されない。blameが必要な場合は、mini.diffだけ使ってgitsigns.nvimの`blame_line`機能だけ残すという部分採用も可能や。
:::

## ステップ4: MiniMaxの設定を参考に整理する

ここまで段階的に移行してきたら、設定が少し散らかってきたはずや。そこでMiniMaxの設定ファイルを参考に、整理・統一していく：

```bash
# MiniMaxの設定を参照しながら自分の設定を整理
cat ~/.config/nvim-minimax/plugin/30_mini.lua
```

MiniMaxは全mini.nvimモジュールの「推奨設定」が書かれた参照実装や。自分の設定と見比べながら、改善できる部分を取り込んでいく。

## 移行しない選択肢

これも正当な選択肢や。

**「mini.aiとmini.surroundだけ追加して、後はlazy.nvim + telescope継続」** という構成は完全に成立する。実際にこのスタイルをとっているNeovimユーザーは多い。

mini.aiはLazyVimにデフォルト採用されているくらいやから、「mini.nvimを使っていない」ユーザーでもmini.aiだけ使うケースは普通にある。

**段階的移行の最終形は人それぞれ**:

```
# パターンA: mini.nvimで完全統一（MiniMaxスタイル）
vim.pack + mini.* 系 + nvim-lspconfig + nvim-treesitter + conform.nvim

# パターンB: ハイブリッド（現実的な多くの人のアプローチ）
lazy.nvim + mini.ai + mini.surround + mini.diff + telescope + blink.cmp + ...

# パターンC: 最小追加（気に入ったモジュールだけ）
lazy.nvim + mini.ai + mini.comment + （他は今まで通り）
```

どれが正解かはない。自分の作業フローと優先順位に合わせて選べばいい。

## 逆移行も簡単

「試してみたけど合わなかった」という場合は：

```lua
-- lazy.nvimの設定からmoduleを削除するだけ
-- { 'echasnovski/mini.pick', ... }  ← この行を消す
```

各モジュールが独立しているので、外した後に依存関係で他が壊れることはない。これがmini.nvimのモジュール設計の利点でもある。

## 移行の所要時間の目安

| フェーズ | 内容 | 慣れるまでの目安 |
|---|---|---|
| ステップ1 | mini.ai, mini.surround追加 | 数日 |
| ステップ2 | mini.files or mini.pickに切り替え | 1〜2週間 |
| ステップ3 | mini.diff + mini.gitに切り替え | 数日 |
| ステップ4 | 全体整理 | 1〜2週間 |

急がなくていい。1つ1つ試して、快適に使えると確信してから次に進むといいで。

次の章では、どんなに完全にmini.nvimに移行しても**必ず外部プラグインが必要な領域**を正直に解説する。
