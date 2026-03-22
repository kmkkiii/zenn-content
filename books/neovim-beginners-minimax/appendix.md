---
title: "付録: チートシートとトラブルシューティング"
---

## MiniMaxキーマップチートシート

### 基本操作

| キー | 動作 |
|---|---|
| `h j k l` | 左下上右に移動 |
| `w b e` | 単語単位で移動 |
| `0 ^ $` | 行頭・非空白行頭・行末 |
| `gg G` | ファイル先頭・末尾 |
| `Ctrl-d Ctrl-u` | 半画面スクロール |
| `i a o I A O` | インサートモードへ |
| `Esc` | ノーマルモードへ |
| `:w :q :wq :q!` | 保存・終了 |
| `u Ctrl-r` | Undo / Redo |

### 編集

| キー | 動作 |
|---|---|
| `dw dd d$ D` | 単語/行/行末/行を削除 |
| `cw cc c$ C` | 単語/行/行末/行を変更 |
| `yy Y` | 行をコピー |
| `p P` | 後/前に貼り付け |
| `ciw diw yiw` | 単語を変更/削除/コピー |
| `ci" di"` | クォート内を変更/削除 |
| `ci( di(` | 括弧内を変更/削除 |
| `gcc` | 行コメントのトグル |
| `gc{motion}` | 範囲コメントのトグル |
| `.` | 直前の変更を繰り返す |

### mini.surround

| キー | 動作 |
|---|---|
| `saiw"` | 単語をダブルクォートで囲む |
| `saiw(` | 単語を括弧で囲む |
| `sd"` | ダブルクォートを削除 |
| `sr"'` | `"hello"` → `'hello'` |
| `srt<span>` | タグをspanに変更 |

### ファイル・バッファ操作（Leaderキーはスペース）

| キー | 動作 |
|---|---|
| `<Space>ef` | 現在のファイルのディレクトリでエクスプローラを開く |
| `<Space>ed` | 作業ディレクトリでエクスプローラを開く |
| `<Space>ff` | ファイルをファジー検索 |
| `<Space>fb` | バッファをファジー検索 |
| `<Space>fg` | ファイル内テキストを検索（grep） |
| `<Space>fh` | ヘルプを検索 |
| `<Space>fv` | 訪問履歴からファイルを選択 |
| `<Space>fr` | 前回のピッカーを再開 |
| `<Space>bd` | バッファを閉じる |
| `<Space>ba` | 直前のバッファに切り替え |
| `[b ]b` | 前/次のバッファへ |

### ウィンドウ操作

| キー | 動作 |
|---|---|
| `Ctrl-w s` | 水平分割 |
| `Ctrl-w v` | 垂直分割 |
| `Ctrl-w h/j/k/l` | ウィンドウ間を移動 |
| `Ctrl-w =` | ウィンドウを均等分割 |
| `Ctrl-w q` | ウィンドウを閉じる |

### LSP操作

| キー | 動作 |
|---|---|
| `gd` | 定義へジャンプ |
| `K` | ホバー情報を表示 |
| `[d ]d` | 前/次の診断へ移動 |
| `<Leader>lr` | リネーム |
| `<Leader>la` | コードアクション |
| `<Leader>ld` | 診断の詳細を表示 |
| `<Leader>lf` | フォーマット |
| `<Leader>ls` | ソース定義へジャンプ |
| `<Leader>lR` | 参照一覧 |

### Git操作（mini.diff / mini.git）

| キー | 動作 |
|---|---|
| `[h ]h` | 前/次のハンクへ移動 |
| `ghgh` | カーソル位置のハンクを適用 |
| `gHgh` | カーソル位置のハンクをリセット |
| `<Leader>go` | オーバーレイ差分表示を切り替え |
| `<Leader>gs` | カーソル位置のGit情報を表示 |
| `<Leader>gc` | コミット |
| `<Leader>gd` | 差分をdiffで表示 |
| `:Git status` | Git状態を確認 |
| `:Git add %` | 現在ファイルをステージ |

### 検索

| キー | 動作 |
|---|---|
| `/pattern` | 前方検索 |
| `?pattern` | 後方検索 |
| `n N` | 次/前の検索結果 |
| `* #` | カーソル下の単語を検索 |
| `f{char} t{char}` | 行内の文字へジャンプ |

## よくあるトラブルと解決策

### アイコンが文字化けする（□や?が表示される）

**原因**: ターミナルのフォントがNerd Fontに設定されていない

**解決策**:
```bash
# まずNerd Fontをインストール
brew install font-hack-nerd-font  # macOS

# ターミナルの設定でフォントを変更する
# iTerm2: Preferences > Profiles > Text > Font
# WezTerm: ~/.wezterm.lua で font = wezterm.font('Hack Nerd Font')
```

確認コマンド：
```bash
echo "\ue0b0"  # → が表示されれば正常
```

### LSPが動かない

**確認手順**:
```vim
:LspInfo    " LSPの接続状況を確認
```

よくある原因と解決策：

```bash
# 言語サーバーが入っていない
which typescript-language-server  # パスを確認

# インストールし直す
npm install -g typescript-language-server typescript

# パスが通っていない
echo $PATH
```

```vim
" Neovim内でpathを確認
:echo $PATH
```

### Tree-sitterのパースエラーが出る

```vim
:TSUpdate         " 全パーサーを更新
:TSInstall lua    " 特定の言語のパーサーを再インストール

" パーサーの状態確認
:TSInstallInfo
```

### プラグインが動かない / 更新したい

**Neovim 0.12+（vim.pack）の場合：**

```vim
:PackInstall   " プラグインをインストール
:PackUpdate    " プラグインを更新
:PackClean     " 不要なプラグインを削除
```

**Neovim 0.10/0.11（mini.deps）の場合：**

```vim
:lua MiniDeps.update()   " プラグインを更新
:lua MiniDeps.log()      " ログを確認
:lua MiniDeps.clean()    " キャッシュをクリア
```

```bash
# プラグインディレクトリを確認
ls ~/.local/share/nvim-minimax/
```

### 設定を間違えてNeovimが起動しない

```bash
# エラーメッセージを表示しながら起動
NVIM_APPNAME=nvim-minimax nvim --startuptime /tmp/nvim-startup.log
cat /tmp/nvim-startup.log

# 問題のある設定ファイルを特定して修正
vim ~/.config/nvim-minimax/plugin/40_plugins.lua
```

Gitで管理していれば元に戻せる：

```bash
cd ~/.config/nvim-minimax
git diff            # 何を変えたか確認
git checkout -- .   # 全変更を取り消す
```

### スワップファイルの警告が出る

```
E325: ATTENTION
```

別のNeovimプロセスが同じファイルを開いている、または前回クラッシュした時の残骸や。

```vim
" スワップファイルが残っている場合
" → [R]ecover / [D]elete / [Q]uit のどれかを選ぶ
" 復元不要なら D でスワップファイルを削除
```

```bash
# スワップファイルの場所
ls ~/.local/share/nvim-minimax/swap/
```

## さらに学ぶためのリソース

### 組み込みチュートリアル

```vim
:Tutor    " 30分でVimの基本を学べる
:h tutor  " チュートリアルのヘルプ
```

### ヘルプの使い方

Neovimのヘルプは充実しとる。`:h`コマンドを使いこなせるようになると自己解決能力が大幅に上がる：

```vim
:h nvim         " Neovim全般
:h mini.files   " mini.filesのドキュメント
:h mini.pick    " mini.pickのドキュメント
:h lsp          " LSP機能のドキュメント
:h usr_01.txt   " Vimユーザーマニュアル（日本語版もある）
```

### おすすめサイト・リポジトリ

- **MiniMaxリポジトリ**: https://github.com/nvim-mini/MiniMax
- **mini.nvim**: https://github.com/echasnovski/mini.nvim
- **Neovim公式ドキュメント**: https://neovim.io/doc/
- **Vim Adventures**: https://vim-adventures.com/ （ゲームでVimを学ぶ）
- **dotfyle**: https://dotfyle.com/ （プラグイン検索サイト）

### 日本語リソース

- **Vim日本語ドキュメント**: https://vim-jp.org/vimdoc-ja/
- **Zennのvim/neovimタグ**: https://zenn.dev/topics/vim

## MiniMaxからの卒業

この本を読み終えて、Neovimに慣れてきたら、次のステップを考えてみてほしい。

**設定をさらに深く理解する:**

```vim
" MiniMaxの設定ファイルを全部読む
:edit ~/.config/nvim-minimax/plugin/30_mini.lua
```

コメントを読みながら、なぜその設定になっているかを理解していくと、Luaの設定ファイルの読み方が身につく。

**独自のプラグインを試す:**

Dotfyleやawesome-neovimで気になるプラグインを見つけたら、`40_plugins.lua`に追加して試してみよう。

**他の人の設定を参考にする:**

GitHubで「nvim config」や「dotfiles neovim」で検索すると、多くの人の設定が公開されとる。気に入った設定をどんどん取り入れていこう。

---

MiniMaxの旅はここから始まる。最初は難しく感じても、毎日少しずつ使っていれば1〜2週間で基本操作が身につく。1ヶ月後には、Neovimなしの開発が考えられないくらいになっているはずや。

頑張ってな！🎉
