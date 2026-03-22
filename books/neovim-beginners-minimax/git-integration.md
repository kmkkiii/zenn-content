---
title: "Git連携"
---

## MiniMaxのGit機能概要

MiniMaxには2つのmini.nvimモジュールでGit連携が実現されとる：

| モジュール | 役割 |
|---|---|
| `mini.diff` | 差分の表示・ハンク操作（ステージ・リセット） |
| `mini.git` | Gitコマンドの実行・ブランチ情報の表示 |

これらを組み合わせることで、ターミナルに切り替えることなくNeovim内でGit操作が完結できるようになる。

## mini.diffで差分を確認する

ファイルを編集すると、変更箇所がガター（行番号の横）に自動的に表示される：

```
1  │ function greet(name: string) {
2 +│   console.log(`Hello, ${name}!`);  ← 追加（緑色）
3  │ }
```

| 記号 | 意味 |
|---|---|
| `+`（緑） | 追加した行 |
| `-`（赤） | 削除した行（前の行に表示） |
| `~`（青） | 変更した行 |

### ハンクの操作

「ハンク」は差分のまとまりのことや。ファイルが複数箇所変更されているとき、それぞれがハンクになる。

```vim
]h    " 次のハンクへ移動
[h    " 前のハンクへ移動
```

MiniMaxのデフォルトキーマップ（`<Leader>g`プレフィックス）：

```vim
<Leader>gh   " ハンクを表示（Hunk）
<Leader>gH   " ハンクのプレビューを表示
<Leader>gs   " ハンクをステージ（Stage）
<Leader>gr   " ハンクをリセット（Reset）
```

### テキストオブジェクトとしてのハンク

`mini.diff` はテキストオブジェクトも提供してくれとる：

```vim
igh   " ハンクの内容を選択（Inner Hunk）
dgh   " ハンクの変更を削除（Diff Git Hunk）
```

### インラインの詳細差分

ガター表示だけでなく、インラインで変更前後を確認したい場合：

```vim
<Leader>gd   " 現在のハンクの差分を表示
```

## mini.gitでGitコマンドを実行する

`mini.git` は`:Git`コマンドを通じてgitコマンドを直接実行できる：

```vim
:Git status
:Git log --oneline
:Git diff HEAD
:Git blame
:Git add %          " 現在のファイルをステージ
:Git commit
:Git push
:Git pull
```

`%` は現在開いているファイルのパスに展開される。

### ステータスラインでのブランチ表示

MiniMaxの `mini.statusline` と `mini.git` が連携して、現在のブランチ名がステータスラインに表示される：

```
 main  ● 2  src/app.ts  TypeScript  LSP
  ^ブランチ名
```

### よく使う操作のまとめ

```vim
:Git status     " 変更ファイルの確認
:Git add %      " 現在のファイルをステージ
:Git add .      " 全ファイルをステージ
:Git commit     " コミット（エディタでメッセージ入力）
:Git push       " リモートにpush
:Git pull       " リモートからpull
:Git log        " コミット履歴
:Git blame      " 各行のコミット情報を表示
```

## 実践ワークフロー

実際の開発でよく使うワークフローを紹介するで。

### ワークフロー1: 通常のコミット

```vim
" 1. ファイルを編集（変更がガターに表示される）

" 2. 変更内容を確認
:Git diff

" 3. 全体の変更状況を確認
:Git status

" 4. ファイルをステージ
:Git add %        " 現在のファイル
" または
:Git add src/     " ディレクトリ全体

" 5. コミット
:Git commit
" エディタが開くのでコミットメッセージを入力してZZで保存終了
```

### ワークフロー2: ハンク単位のステージ

「ファイルの一部だけをステージしたい」という場面で役立つ：

```vim
" 1. 変更のあるファイルを開く

" 2. ]h でステージしたいハンクに移動

" 3. <Leader>gs でハンクをステージ

" 4. 不要な変更は <Leader>gr でリセット

" 5. :Git commit でコミット
```

### ワークフロー3: コミット履歴を確認する

```vim
" コミット一覧を表示
:Git log --oneline --graph

" 特定のコミットの内容を確認
:Git show abc1234

" 特定のファイルの変更履歴
:Git log --follow src/app.ts
```

## コンフリクトの解決

マージコンフリクトが発生した時の基本的な対処方法や。

コンフリクトが発生したファイルには以下のようなマーカーが入る：

```
<<<<<<< HEAD
  const x = 1;  // 自分の変更
=======
  const x = 2;  // 相手の変更
>>>>>>> feature-branch
```

Neovimでの解決手順：

```vim
" 1. コンフリクトファイルを開く
:Git status   " コンフリクトファイルの確認

" 2. マーカーを探す
/<<<<<<   " コンフリクトマーカーを検索

" 3. 手動でコードを編集してコンフリクトを解消
" マーカーを含む不要な行を削除して、正しいコードを残す

" 4. 解消したファイルをステージ
:Git add %

" 5. マージコミット
:Git commit
```

:::message
コンフリクト解消は初心者には難しく感じることがある。`mini.diff`の差分表示を活用して、どちらの変更を取り込むか慎重に判断してな。不安な時は`git mergetool`でGUIのマージツールを使うのも選択肢や。
:::

## Gitの操作をもっと便利にする設定

`plugin/20_keymaps.lua` に以下のようなキーマップを追加すると、よく使うGit操作に素早くアクセスできる：

```lua
-- plugin/20_keymaps.lua に追記
local map = vim.keymap.set

-- Gitキーマップ
map('n', '<Leader>ga', ':Git add %<CR>', { desc = 'Git add current file' })
map('n', '<Leader>gc', ':Git commit<CR>', { desc = 'Git commit' })
map('n', '<Leader>gp', ':Git push<CR>', { desc = 'Git push' })
map('n', '<Leader>gl', ':Git log --oneline<CR>', { desc = 'Git log' })
map('n', '<Leader>gst', ':Git status<CR>', { desc = 'Git status' })
```

## まとめ

MiniMaxのGit連携の主なポイント：

- **mini.diff**: ガターでリアルタイム差分確認、ハンク単位の操作
- **mini.git**: `:Git`コマンドで全てのgit操作が可能
- **mini.statusline連携**: ブランチ名が常に見える

次の章では、MiniMaxの設定をカスタマイズして、自分だけの環境を作っていく方法を学ぶで！
