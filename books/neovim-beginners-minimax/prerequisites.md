---
title: "環境準備"
---

## 必要なもの

MiniMaxを使ったNeovim環境を構築するために必要なものを準備しよう。

- **Neovim** 0.10以上（0.11以上を推奨）
- **Git**（MiniMaxのクローンに使う）
- **ターミナル**（Nerd Font対応のものを推奨）
- **ripgrep**（オプション、ファイル内検索の高速化）

## Neovimのインストール

### macOS

Homebrewを使うのが一番簡単や：

```bash
brew install neovim
```

最新の安定版を確認：

```bash
nvim --version
# NVIM v0.11.0
# Build type: Release
# ...
```

### Ubuntu / Debian

パッケージマネージャ経由だと古いバージョンが入ることがあるので、公式のAppImageか、[Neovim GitHub Releases](https://github.com/neovim/neovim/releases)から直接インストールするのがおすすめや。

**AppImageを使う方法：**

```bash
curl -LO https://github.com/neovim/neovim/releases/latest/download/nvim-linux-x86_64.appimage
chmod u+x nvim-linux-x86_64.appimage
sudo mv nvim-linux-x86_64.appimage /usr/local/bin/nvim
```

**aptで入れる場合（Ubuntu 24.04以降は0.9以上が入る）：**

```bash
sudo apt update
sudo apt install neovim
nvim --version  # バージョンを確認
```

0.10未満の古いバージョンが入った場合は、AppImageかSnap経由でのインストールを試してな：

```bash
sudo snap install nvim --classic
```

### Windows

**Scoopを使う方法（推奨）：**

```powershell
scoop install neovim
```

**wingetを使う方法：**

```powershell
winget install Neovim.Neovim
```

:::message
WindowsでNeovimを使う場合、WSL2（Windows Subsystem for Linux）上のUbuntuにインストールする方法が最もトラブルが少なくておすすめや。WSL2 + Windows Terminal + Neovimは相性が良い。
:::

## ターミナルの準備

MiniMaxはアイコン表示に **Nerd Font**（アイコングリフを含む特殊フォント）を使う。対応したターミナルとフォントの設定が必要や。

### 推奨ターミナル

| ターミナル | 対応OS | 特徴 |
|---|---|---|
| [WezTerm](https://wezfurlong.org/wezterm/) | macOS/Linux/Windows | Lua設定、GPU加速、多機能 |
| [Alacritty](https://alacritty.org/) | macOS/Linux/Windows | 超高速、シンプル |
| [Kitty](https://sw.kovidgoyal.net/kitty/) | macOS/Linux | 高機能、タイル分割 |
| [iTerm2](https://iterm2.com/) | macOS | macOS定番、機能豊富 |
| Windows Terminal | Windows | Microsoft公式、WSL2対応 |

### Nerd Fontのインストール

**macOS（Homebrewで一括インストール）：**

```bash
# フォント一覧を確認
brew search nerd-font

# 人気のフォントをインストール（いくつか例）
brew install font-hack-nerd-font
brew install font-jetbrains-mono-nerd-font
brew install font-fira-code-nerd-font
```

**Linux：**

```bash
# 手動でダウンロードする場合
mkdir -p ~/.local/share/fonts
cd ~/.local/share/fonts

# JetBrains Mono Nerd Fontを例に
curl -LO https://github.com/ryanoasis/nerd-fonts/releases/latest/download/JetBrainsMono.zip
unzip JetBrainsMono.zip
fc-cache -fv
```

**Windows：**

[Nerd Fonts公式サイト](https://www.nerdfonts.com/font-downloads)からダウンロードして、フォントファイルを右クリック→「インストール」で入る。

### ターミナルにフォントを設定する

**WezTermの場合（`~/.wezterm.lua`）：**

```lua
local wezterm = require 'wezterm'
return {
  font = wezterm.font('JetBrainsMono Nerd Font'),
  font_size = 14.0,
}
```

**iTerm2の場合：**

`Preferences` → `Profiles` → `Text` → `Font` で「JetBrainsMono Nerd Font」を選択。

**Windows Terminalの場合（`settings.json`）：**

```json
{
  "profiles": {
    "defaults": {
      "font": {
        "face": "JetBrainsMono Nerd Font"
      }
    }
  }
}
```

:::message
フォントが正しく設定されているか確認したい時は、ターミナルで以下を試してな：
```bash
echo "\ue0b0 \uf015 \ue606"
```
矢印やホームアイコンが表示されれば正常や。豆腐（□）が出る場合はフォントの設定を見直してな。
:::

## ripgrepのインストール（推奨）

ripgrepはファイル内の高速全文検索ツールや。MiniMaxの`mini.pick`（ファジーファインダー）と連携して、プロジェクト内のテキスト検索がサクサク動くようになる。

```bash
# macOS
brew install ripgrep

# Ubuntu/Debian
sudo apt install ripgrep

# 確認
rg --version
# ripgrep 14.x.x
```

## NVIM_APPNAMEの概念を理解しよう

Neovimには `NVIM_APPNAME` という環境変数があって、これを指定することで設定ディレクトリを切り替えられる。

```bash
# 通常のNeovim（~/.config/nvimを使う）
nvim

# MiniMax用Neovim（~/.config/nvim-minimaxを使う）
NVIM_APPNAME=nvim-minimax nvim
```

これにより、**今使っているNeovimの設定を一切変えずに**、別の設定でNeovimを試せる。既存の設定がある人も安心してMiniMaxを試せるわけや。

### シェルエイリアスを設定しておこう

毎回 `NVIM_APPNAME=nvim-minimax` と入力するのは面倒なので、エイリアスを設定しておくと便利や。

**~/.bashrc または ~/.zshrc に追記：**

```bash
# MiniMax版Neovimのエイリアス
alias nvm='NVIM_APPNAME=nvim-minimax nvim'
```

設定を反映：

```bash
source ~/.zshrc  # zshの場合
source ~/.bashrc # bashの場合
```

以降は `nvm` コマンドでMiniMaxを使ったNeovimが起動するようになる。

:::message alert
エイリアス名は自由に決めてええで。`mm`とか`mxvim`とか、打ちやすいものにしてな。`vim`や`nvim`コマンドを上書きするエイリアスは避けた方が無難や。
:::

## 準備完了チェックリスト

次の章に進む前に確認してな：

- [ ] `nvim --version` でv0.10以上が表示される
- [ ] `git --version` でGitが使える
- [ ] ターミナルにNerd Fontが設定されている
- [ ] （推奨）`rg --version` でripgrepが使える
- [ ] `NVIM_APPNAME=nvim-minimax nvim --version` が動く（エラーが出ない）

全部チェックできたら、次の章でMiniMaxをセットアップしよう！
