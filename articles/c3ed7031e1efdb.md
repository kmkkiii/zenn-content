---
title: 'macOS向けの新しいウィンドウマネージャーRift'
emoji: '🪟'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['Rift', 'macOS', 'windowmanager']
published: false
---

X で見かけた macOS 向けのウィンドウマネージャー Rift を試してみます。

ウィンドウマネージャーとしては yabai や Aerospace の名前をよく見かけます。
私も Aerospace を使用しており、モニターごとに``

仕事では Mac を使用しており、Raycast の Window Management 機能で事足りている状態ではありました。

ただ、最近 Omarchy を古い Intel Mac に入れて触っている内にタイリングウィンドウマネージャーの良さを改めて感じたので、Mac でもタイリングウィンドウマネージャーを導入したいと考えているところでした。

最近の MacOS(Sequoia 以降？)でも、タイル表示ができるようになっています。
https://support.apple.com/ja-jp/guide/mac-help/mchl9674d0b0/mac

よく目にするウィンドウマネージャーとしては yabai や Aerospace があります。

複数の外部モニターを接続していると、スリープからの復帰時に各モニターに配置していたウィンドウやアプリがぐちゃぐちゃになってしまうことがあります。
そのようなストレスを解消するために Aerospace の`workspace-to-monitor-force-assignment`で特定のワークスペースをモニターに割り当てるようにしていました。

いいところ

- 設定ファイルのホットリロードがある
- Mission Control の設定を有効にした状態で問題なく利用できる
- 高いパフォーマンス

今後に期待しているところ

- 複数モニターへの割り当て

Aerospace の`workspace-to-monitor-force-assignment`相当の機能が追加されたら本格的に移行を検討したいです。
