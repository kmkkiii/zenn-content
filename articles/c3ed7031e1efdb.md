---
title: 'macOS向けの新しいタイル型ウィンドウマネージャーRift'
emoji: '🪟'
type: 'idea' # tech: 技術記事 / idea: アイデア
topics: ['Rift', 'macOS']
published: true
---

最近、古い Intel MacBook Pro に Omarchy を入れて触っている内にタイル型ウィンドウマネージャーの良さがわかってきたので、仕事でつかっている macOS でも導入したいと考えるようになりました。

一応 Aerospace は既に導入してありますが、起動時ディスプレイごとにアプリを配置する機能がメインみたいな使い方をしており、ウィンドウ操作は基本的に Raycast やトラックパッドで行っている状態でした。

今回は X で見かけた macOS 向けの新しいウィンドウマネージャー **Rift** を試してみます。

## Rift とは

https://github.com/acsandmann/rift

yabai、Aerospace に次いで登場した macOS 向けのタイル型ウィンドウマネージャーで以下の特徴があります。

- Rust 製
- 複数レイアウトスタイルに対応
  - タイル型
  - BSP
- パフォーマンスを最優先事項として開発されている
  - 公開されているアクセシビリティ API ではなく、非公開 API を使用
- マウスホバーしたウィンドウへ自動的にフォーカスする
- ホットリロード可能な設定
- トラックパッドのジェスチャーでワークスペース切り替えが可能
- SIP 無効化が不要
  - yabai は一部 SIP を無効にすることでスペースの操作を可能にしている
- 「ディスプレイごとに別々のスペースを使用する」機能が有効となっている状態で動作する
  - Aerospace は無効化を推奨
  - https://nikitabobko.github.io/AeroSpace/guide#a-note-on-displays-have-separate-spaces
- CLI や公開 mach ポートを用いた高いカスタマイズ性

## 使ってみた感想

特徴に挙げたような以下の機能は Aerospace になかったので便利でした。

> - レイアウトスタイルがタイル型と BSP から選べる
> - マウスホバーしたウィンドウへ自動的にフォーカスする
> - ホットリロード可能な設定
> - トラックパッドのジェスチャーでワークスペース切り替えが可能

一方で、以下の挙動は現状許容できなかったため、今回 Aerospace からの移行は見送りました。

- ディスプレイにそれぞれ独立したワークスペースのセットが割り当てられる
- Rift を有効化した際にすべてのウィンドウが単一のディスプレイに表示されてしまう
- Aerospace の`workspace-to-monitor-force-assignment`のような特定のワークスペースをモニターに割り当てることができない

上記の挙動は Rift が提供している CLI (rift-cli)や Hammerspoon を使うことで解決できるとは思います。
MacBook 単体や単一のディスプレイで使う分には申し分ないのですが、複数ディスプレイ環境に対するサポートはまだまだこれからのようです(自分の設定方法がまちがっている可能性も高いですが...)。

Mac のスリープ復帰時に各ディスプレイに配置していたウィンドウ配置がぐちゃぐちゃになってしまう問題はすでに解消しているかもしれませんが、Aerospace の`workspace-to-monitor-force-assignment`相当の機能は欲しい。

## 余談

最近の MacOS(Sequoia 以降)では、タイル表示ができるようになっていました。
https://support.apple.com/ja-jp/guide/mac-help/mchl9674d0b0/mac
