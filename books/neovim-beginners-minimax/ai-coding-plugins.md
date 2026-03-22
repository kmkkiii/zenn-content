---
title: "ボーナス: AIコーディングプラグイン"
---

## Neovimでもっと便利にAIを使う

ここまでMiniMaxを使ったNeovimの基本をマスターしてきた。このボーナスチャプターでは、最近急速に発展しているAIコーディングプラグインを紹介するで。

AIコーディング支援の選択肢は多い。まず全体像を掴もう：

| プラグイン | スター数 | 特徴 | 向いている人 |
|---|---|---|---|
| **copilot.lua** | ★12k+ | インライン補完、最も安定 | GitHub Copilot契約者 |
| **avante.nvim** | ★17.7k | Cursor風サイドバー、最人気 | CursorからNeovimに移行したい人 |
| **codecompanion.nvim** | ★6.2k | Vim-native、無料モデル対応 | Vimらしい使い方が好みの人 |
| **sidekick.nvim** | ★2.4k | NES + CLIツール統合、folke作 | Claude Code/aider ユーザー |

## copilot.lua - インライン補完の定番

GitHub Copilotを使うなら、公式の `copilot.vim` より `copilot.lua` の方が断然おすすめや。純粋なLua実装で、Neovimとの統合が自然でパフォーマンスも良い。

### インストール

```lua
-- plugin/40_plugins.lua に追記
vim.pack.add('https://github.com/zbirenbaum/copilot.lua')

require('copilot').setup({
  suggestion = {
    enabled = true,
    auto_trigger = true,   -- 入力中に自動で提案
    keymap = {
      accept = '<Tab>',      -- Tabで提案を受け入れ
      accept_word = '<C-Right>',
      accept_line = '<C-Down>',
      next = '<M-]>',        -- 次の提案
      prev = '<M-[>',        -- 前の提案
      dismiss = '<C-e>',     -- 提案を閉じる
    },
  },
  panel = {
    enabled = true,
    auto_refresh = false,
    keymap = {
      open = '<M-CR>',       -- パネルを開く
    },
  },
  filetypes = {
    markdown = true,
    yaml = true,
  },
})
```

:::message alert
**mini.completionとの競合に注意**

copilot.luaのインライン提案（ghost text）とmini.completionの補完ポップアップは、どちらも`<Tab>`を使おうとして競合することがある。

以下のように`<Tab>`以外のキーを割り当てるか、`auto_trigger = false`にして手動トリガーにするといい：

```lua
keymap = {
  accept = '<M-l>',  -- Alt+l で受け入れ
}
```
:::

### 認証

初回は認証が必要や：

```vim
:Copilot setup
" ブラウザでGitHubのデバイスコード認証が開く
```

### チャット機能を追加する（CopilotChat.nvim）

インライン補完に加えて、チャットUIも欲しい場合は `CopilotChat.nvim` を追加：

```lua
vim.pack.add('https://github.com/CopilotC-Nvim/CopilotChat.nvim')

require('CopilotChat').setup({
  model = 'claude-sonnet-4-5',  -- 使用するモデル
  window = {
    layout = 'vertical',        -- サイドバーとして表示
    width = 0.4,
  },
})

-- キーマップ
vim.keymap.set('n', '<Leader>cc', ':CopilotChat<CR>', { desc = 'CopilotChat' })
vim.keymap.set('v', '<Leader>cc', ':CopilotChat<CR>', { desc = 'CopilotChat with selection' })
```

## avante.nvim - Cursor風AIサイドバー

avante.nvimはCursorのAI機能をNeovimで再現しようとしとる最も人気のプラグインや（★17.7k）。複数のLLMプロバイダに対応していて、コードの生成・修正をサイドバーで操作できる。

### インストール

avante.nvimはRustのコンポーネントをビルドするので、Cargoが必要や：

```bash
# Rustをインストールしていない場合
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

```lua
-- plugin/40_plugins.lua に追記
vim.pack.add({
  src = 'https://github.com/yetone/avante.nvim',
  build = 'make',   -- バイナリをビルド（:PackInstall時に実行）
})

-- 依存プラグイン
vim.pack.add('https://github.com/stevearc/dressing.nvim')
vim.pack.add('https://github.com/MunifTanjim/nui.nvim')
vim.pack.add('https://github.com/MeanderingProgrammer/render-markdown.nvim')

require('avante').setup({
  provider = 'claude',   -- または 'openai', 'gemini', 'ollama'
  claude = {
    endpoint = 'https://api.anthropic.com',
    model = 'claude-sonnet-4-5',
    api_key_name = 'ANTHROPIC_API_KEY',  -- 環境変数名
    max_tokens = 4096,
  },
})
```

### APIキーの設定

```bash
# ~/.zshrc または ~/.bashrc に追記
export ANTHROPIC_API_KEY='your-api-key-here'
export OPENAI_API_KEY='your-api-key-here'
```

### 使い方

```vim
<Leader>aa   " avanteサイドバーを開く/閉じる（Ask Avante）
<Leader>ae   " コードを説明させる（Explain）
<Leader>af   " コードを修正させる（Fix）
<Leader>ar   " コードをリファクタする（Refactor）
```

ビジュアルモードでコードを選択してから`<Leader>aa`を押すと、選択したコードについてAIに質問できる：

```
選択: function getUserById(id) { ... }
<Leader>aa
> このコードにTypeScriptの型注釈を追加して
```

avanteはdiffとして変更を提案してきて、`<Leader>ay`で適用、`<Leader>an`で却下できる。

## codecompanion.nvim - Vim-native AI支援

codecompanion.nvimは「Vimらしさ」を大切にした設計のAIプラグインや（★6.2k）。無料のGitHub Modelsエンドポイントが使えるので、APIキーなしでも試せる。

### インストール

```lua
-- plugin/40_plugins.lua に追記
vim.pack.add('https://github.com/olimorris/codecompanion.nvim')
vim.pack.add('https://github.com/nvim-lua/plenary.nvim')  -- 依存

require('codecompanion').setup({
  strategies = {
    chat = {
      adapter = 'anthropic',
    },
    inline = {
      adapter = 'anthropic',
    },
  },
  adapters = {
    anthropic = function()
      return require('codecompanion.adapters').extend('anthropic', {
        env = {
          api_key = 'ANTHROPIC_API_KEY',
        },
        schema = {
          model = {
            default = 'claude-sonnet-4-5',
          },
        },
      })
    end,
    -- GitHub Models（無料！）
    github_models = function()
      return require('codecompanion.adapters').extend('openai', {
        env = {
          api_key = 'GITHUB_TOKEN',
        },
        url = 'https://models.inference.ai.azure.com/chat/completions',
        schema = {
          model = {
            default = 'gpt-4o',
          },
        },
      })
    end,
  },
})

-- キーマップ
vim.keymap.set({ 'n', 'v' }, '<Leader>ic', ':CodeCompanionChat<CR>', { desc = 'CodeCompanion Chat' })
vim.keymap.set({ 'n', 'v' }, '<Leader>ii', ':CodeCompanionInline<CR>', { desc = 'CodeCompanion Inline' })
```

### GitHub Modelsで無料で使う

```bash
# GitHubトークンを設定（無料）
export GITHUB_TOKEN='ghp_your_token_here'
```

GitHub Personal Access Tokenがあれば、`gpt-4o`や`claude-sonnet`が無料で使える。

### 使い方

```vim
:CodeCompanionChat            " チャット画面を開く
:CodeCompanionInline          " インラインでAIに編集させる
:CodeCompanion /explain       " コードを説明
:CodeCompanion /fix           " バグを修正
:CodeCompanion /lsp           " LSPエラーを修正
:CodeCompanion /tests         " テストを生成
```

codecompanionの特徴は`:cc`のような短い略語でコマンドが使えること、そして`@`でファイルや`#`でシンボルをコンテキストに追加できる点や。

## folke/sidekick.nvim - NES + CLIツール統合

sidekick.nvimはfolke（lazy.nvim, which-key.nvimの作者）が2025年後半にリリースした注目のプラグインや（★2.4k）。

**2つの機能が特徴：**

1. **Next Edit Suggestions（NES）**: GitHub Copilotの最新機能。コードを変更した後、それに関連する他の変更箇所を自動で提案してくれる。
2. **CLIツール統合**: Claude Code、aider、Gemini CLIなどのAI CLIツールをNeovim内のターミナルで管理できる。

### インストール

```lua
-- plugin/40_plugins.lua に追記
vim.pack.add('https://github.com/folke/sidekick.nvim')

require('sidekick').setup({
  -- NES（Next Edit Suggestions）の設定
  nes = {
    enabled = true,
  },
  -- CLIツールの設定
  tools = {
    {
      name = 'claude',
      cmd = 'claude',            -- Claude Code
      pattern = '.*',
    },
    {
      name = 'aider',
      cmd = 'aider',             -- aider
    },
  },
})

-- キーマップ
vim.keymap.set('n', '<Leader>sk', ':Sidekick<CR>', { desc = 'Sidekick' })
```

### NESの使い方

NESはGitHub Copilotのサブスクリプションが必要や。有効化すると、コードを変更した後に関連する変更提案が自動で表示される：

```typescript
// getUserById(id: string) をリネームして getUserByUserId に変更すると
// → 同じ関数を呼び出している全ての箇所に変更提案が表示される

// 提案を確認してTab/Shift-Tabで適用/却下
```

### CLIツールとの連携

sidekick.nvimの「CLIツール統合」機能は、Claude CodeやaiderをNeovimの管理された端末から起動できる：

```vim
:Sidekick claude   " Claude Codeを起動
:Sidekick aider    " aiderを起動
```

Neovimのウィンドウ内にターミナルが開き、tmuxやzellij経由でセッションが維持される。Neovimを閉じても作業を再開できる。

## ターミナルモードでAI CLIを使う

プラグインを使わずに、Neovim組み込みのターミナルでAI CLIを使う方法もある：

```vim
:terminal    " ターミナルモードを開く
:split | terminal  " 水平分割でターミナルを開く
:vsplit | terminal " 垂直分割でターミナルを開く
```

ターミナルモードでの操作：

```vim
i          " ターミナルへの入力を開始
Ctrl-\n    " ターミナルからノーマルモードに戻る（Ctrl-\, Ctrl-n）
```

```bash
# ターミナル内で
claude "src/app.tsのバグを修正して"
aider src/app.ts
```

キーマップを設定しておくと便利や：

```lua
-- plugin/20_keymaps.lua に追記
vim.keymap.set('n', '<Leader>tt', ':vsplit | terminal<CR>i', { desc = 'Open terminal' })
```

## 各ツールの比較と選び方

```
GitHub Copilot契約がある
  ├── インライン補完だけでいい    → copilot.lua
  ├── チャットも使いたい          → copilot.lua + CopilotChat.nvim
  └── NES + CLIツール管理         → sidekick.nvim（+ copilot.lua）

GitHub Copilot契約なし / 複数のLLMを使いたい
  ├── 無料でまず試したい          → codecompanion.nvim (GitHub Models)
  ├── Cursor風UIが好み            → avante.nvim
  ├── Vimらしく使いたい           → codecompanion.nvim
  └── AIはCLIツールで十分         → sidekick.nvim or ターミナルモード
```

:::message
どれを選ぶかで迷ったら、まず`codecompanion.nvim`をGitHub Modelsで試してみるのがおすすめや。無料で使えて設定もシンプル、使い続けるなら有料プロバイダに切り替えやすい。
:::

## lambdalisue作のNeovimプラグインについて

日本のVimコミュニティで有名なlambdalisue氏（fern.vim, gina.vimの作者）は、denops.vimというDenoランタイム上で動くプラグインフレームワークを使ったAI関連プラグインも開発しとる。

- **nvim-aibo**: AIアシスタント連携
- **butler.vim**: ChatGPT/Ollama連携

Denoが必要という依存関係があるため今回のメイン紹介からは外したけど、denopsエコシステムを既に使っとる人には選択肢に入れてみてもええで。

## まとめ

Neovimには今や充実したAIコーディング支援環境が揃っとる。

- **インライン補完**: copilot.lua
- **AIサイドバー（Cursor風）**: avante.nvim
- **Vim-nativeなAI支援**: codecompanion.nvim
- **NES + CLIツール管理**: sidekick.nvim

自分の使い方に合ったものを選んで、AIとの協業をより快適にしていこう！
