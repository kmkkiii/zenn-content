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

ハンクの操作は `gh` / `gH` オペレータで行う：

```vim
gh   " ハンクを適用するオペレータ（Apply hunk）
gH   " ハンクをリセットするオペレータ（Reset hunk）
```

オペレータなので、モーションやテキストオブジェクトと組み合わせて使う：

```vim
ghgh   " カーソル位置のハンクを適用（gh + ghテキストオブジェクト）
gHgh   " カーソル位置のハンクをリセット
ghip   " 段落内のハンクをまとめて適用
gHG    " カーソルからバッファの末尾までのハンクをリセット
```

### テキストオブジェクトとしてのハンク

`mini.diff` はテキストオブジェクトも提供してくれとる：

```vim
igh   " ハンクの内容を選択（Inner Hunk）
agh   " ハンク全体を選択（Around Hunk）
```

### インラインの詳細差分（オーバーレイ）

ガター表示だけでなく、インラインで変更前後を並べて確認したい場合：

```vim
<Leader>go   " オーバーレイ表示を切り替え
<Leader>gd   " 変更全体をdiffとして別タブで表示
<Leader>gD   " 現在のバッファの変更をdiffで表示
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
" :Gitコマンド
:Git status     " 変更ファイルの確認
:Git add %      " 現在のファイルをステージ
:Git add .      " 全ファイルをステージ
:Git push       " リモートにpush
:Git pull       " リモートからpull
:Git blame      " 各行のコミット情報を表示
```

MiniMaxの`<Leader>g`キーマップ（よく使うものを厳選）：

```vim
<Leader>gc   " コミット（Git commit）
<Leader>gC   " コミット（アメンド）
<Leader>gd   " 変更をdiffで表示（全体）
<Leader>gD   " 変更をdiffで表示（現在のバッファ）
<Leader>ga   " ステージ済みの変更をdiffで確認（Added diff）
<Leader>gl   " コミットログを表示
<Leader>gL   " 現在のバッファのログを表示
<Leader>go   " オーバーレイ表示を切り替え（mini.diff）
<Leader>gs   " カーソル位置のGit情報を表示（mini.git）
```

:::message
`<Leader>gs` はハンクのステージではなく、**カーソル位置のGit情報を表示する**コマンドや（`MiniGit.show_at_cursor()`）。ハンクのステージは `ghgh`（`gh` オペレータ）で行う。
:::

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

" 3. ghgh でハンクをステージ（gh オペレータ + gh テキストオブジェクト）

" 4. 不要な変更は gHgh でリセット

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

MiniMaxには便利なGitキーマップが既に多数定義されとる。追加したい場合は `plugin/20_keymaps.lua` に書く：

```lua
-- plugin/20_keymaps.lua に追記
local nmap = function(lhs, rhs, desc)
  vim.keymap.set('n', lhs, rhs, { desc = desc })
end

-- 追加でよく使うGit操作（既存の<Leader>g*と被らないように）
nmap('<Leader>gp', '<Cmd>Git push<CR>',   'Push')
nmap('<Leader>gP', '<Cmd>Git pull<CR>',   'Pull')
```

## まとめ

MiniMaxのGit連携の主なポイント：

- **mini.diff**: ガターでリアルタイム差分確認、ハンク単位の操作
- **mini.git**: `:Git`コマンドで全てのgit操作が可能
- **mini.statusline連携**: ブランチ名が常に見える

次の章では、MiniMaxの設定をカスタマイズして、自分だけの環境を作っていく方法を学ぶで！
