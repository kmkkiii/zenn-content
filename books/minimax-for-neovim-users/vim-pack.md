---
title: "vim.packとmini.depsの関係"
---

## パッケージマネージャの変遷

Neovimのプラグイン管理の歴史を簡単に振り返ろう：

```
Vim時代:  pathogen → Vundle → vim-plug
Neovim時代: packer.nvim（廃止）→ lazy.nvim（現在の主流）
                               → mini.deps（mini.nvimエコシステム）
                               → vim.pack（Neovim 0.12〜 組み込み）
```

2026年3月、Neovim 0.12がリリースされ（執筆時点でリリース直前）、`vim.pack`という組み込みパッケージマネージャが使えるようになった。

## vim.packとは

`vim.pack`はNeovimのコアに組み込まれたLuaベースのパッケージマネージャや。外部プラグインが不要になる。

```lua
-- vim.packの基本的な使い方（Neovim 0.12+）

-- プラグインの追加（GitHubのフルURLが必要）
vim.pack.add('https://github.com/echasnovski/mini.nvim')

-- バージョン指定
vim.pack.add({
  src = 'https://github.com/neovim/nvim-lspconfig',
  version = 'v1.0.0',
})

-- ビルドコマンド付き
vim.pack.add({
  src = 'https://github.com/nvim-treesitter/nvim-treesitter',
  build = ':TSUpdate',
})
```

管理コマンド：

```vim
:PackInstall   " プラグインをインストール
:PackUpdate    " プラグインを更新
:PackClean     " 不要なプラグインを削除
```

## mini.depsとの関係

**mini.depsはvim.packに「負けた」のではなく、役割を引き継いでもらった形や。**

echasnovski自身がこの移行を歓迎している。MiniMaxの0.12設定はmini.depsからvim.packに切り替わっとる。

```lua
-- MiniMaxの0.11設定（mini.deps）
MiniDeps.add({ source = 'neovim/nvim-lspconfig' })
MiniDeps.later(function()
  MiniDeps.add({ source = 'nvim-treesitter/nvim-treesitter' })
end)

-- MiniMaxの0.12設定（vim.pack）
vim.pack.add('https://github.com/neovim/nvim-lspconfig')
vim.pack.add({
  src = 'https://github.com/nvim-treesitter/nvim-treesitter',
  build = ':TSUpdate',
})
```

mini.depsはNeovim 0.12リリース後、メンテナンスのみで新機能開発が終了予定。今後Neovim 0.12以降を使う人はvim.packを使うことになる。

## vim.packとlazy.nvimの比較

vim.packが登場したことで「lazy.nvimはオワコン？」という疑問が浮かぶかもしれない。答えはNOや。

| | vim.pack | lazy.nvim |
|---|---|---|
| 提供元 | Neovim本体 | サードパーティ（folke作） |
| インストール | 不要 | 必要 |
| URLスタイル | フルURL必須 | `user/repo`でOK |
| lazy-loading | **なし** | 充実（自動・条件指定等） |
| キャッシュ・最適化 | なし | bytecodeキャッシュ等 |
| 設定UI | なし | `:Lazy`でGUI管理 |
| プロファイリング | なし | `:Lazy profile`で起動時間分析 |

**lazy-loadingの有無が大きな違い**。lazy.nvimはプラグインをコマンド実行時やファイルタイプ検出時に遅延ロードすることで、起動時間を短縮できる。

```lua
-- lazy.nvimでのlazy-loading例
{
  'nvim-telescope/telescope.nvim',
  cmd = 'Telescope',           -- :Telescope コマンド実行時にロード
  keys = { '<leader>ff', '<leader>fg' },  -- キーマップ実行時にロード
}
```

vim.packにはこの機能がない。プラグイン数が多くて起動時間が気になる場合は、lazy.nvimの方が現実的な選択や。

## 「プラグインマネージャ要らずの時代」が来つつある

vim.packの登場が示す方向性は興味深い。

Neovimは少しずつ「かつてはプラグインが必要だった機能」をコアに取り込んでいる：

- **vim.loader**（0.9〜）: モジュールキャッシュ
- **vim.lsp**（0.5〜）: ネイティブLSPクライアント
- **vim.treesitter**（0.7〜）: ネイティブTree-sitterサポート
- **vim.pack**（0.12〜）: パッケージマネージャ

これはVSCodeが機能をコアに持つアプローチとは違う——Neovimはシンプルなコアと拡張可能なエコシステムのバランスを保ちながら、基盤機能を強化していく方向性だ。

mini.nvimエコシステムはこの方向性と相性が良い。vim.packが「プラグイン管理はコアで」という標準を作ることで、プラグイン本体（mini.nvimモジュール群）の価値がより際立つ。

## 実際のところ: 誰がvim.packを使うべきか

```
vim.packを選ぶべき人:
- Neovim 0.12+ を使っていて設定をシンプルにしたい
- mini.nvimエコシステムで統一していて、起動時プラグイン数が少ない
- lazy-loadingが不要（そもそもプラグイン数が少なければ不要）
- 外部依存を最小化したい

lazy.nvimを続けるべき人:
- Neovim 0.10/0.11を使っている（vim.packが使えない）
- プラグイン数が多く、lazy-loadingで起動時間を最適化している
- `:Lazy`のGUI管理やプロファイリング機能を活用している
- 既存設定が安定していて変える理由がない
```

## バージョン別まとめ

| Neovimバージョン | 推奨パッケージマネージャ | MiniMaxの設定 |
|---|---|---|
| 0.10 | mini.deps | MiniMax 0.10設定を使用 |
| 0.11 | mini.deps | MiniMax 0.11設定を使用 |
| 0.12+ | **vim.pack** | MiniMax 0.12設定を使用（mini.deps→vim.packに移行済み） |

次は最終章。mini.nvimエコシステムを選ぶべき人・そうでない人を正直にまとめるで。
