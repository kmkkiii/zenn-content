---
title: "wicked_pdfの後継になる？FerrumでHTMLからPDFを生成する" # 記事のタイトル
emoji: "📄" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Rails", "PDF", "Ferrum", "wickedpdf", "wkhtmltopdf"] # タグ。["markdown", "rust", "aws"]のように指定する
publication_name: "mofmof_inc"
published: true # 公開設定（falseにすると下書き）
---

# はじめに

:::message
この記事は「[mofmof Advent Calendar 2023](https://qiita.com/advent-calendar/2023/mofmof)」22 日目の記事です。
:::

`wicked_pdf`を使わない場合の代替案として`Ferrum`という gem を使ってみました。

元々は PDF 生成機能を実装するために`wicked_pdf`を使おうとしていましたが、
依存している`wkhtmltopdf`が今年に入ってアーカイブされていたことを知りました。
内部的に利用している`QT Webkit`レンダリングエンジンのメンテナンスが止まったことで、`wkhtmltopdf`もメンテナンスが継続できなくなってしまったようです。

# wicked_pdf の現状

`wicked_pdf`は`wkhtmltopdf-binary`と併せて利用しますが、`wkhtmltopdf-binary`のメンテナンスが滞っているようです。

willnet さんが M1 Mac(arm64)対応や debian12(bookworm)対応をしてくださった PR が出されていますが、リポジトリは管理されている様子がなく、マージされる気配も感じられません。

https://github.com/zakird/wkhtmltopdf_binary_gem/pull/145

M1 Mac(arm64)や debian12 が使われている Ruby 公式イメージを使用していて`wicked_pdf`を使いたい場合は、この PR を元に公開された`wkhtmltopdf-binary-ng`という別の gem を使うのが現状良さそうです。

```:Gemfile
gem 'wkhtmltopdf-binary-ng'
```

# PDF 生成どうするか

選択肢としては、ヘッドレスブラウザ等で HTML を PDF に変換できる`Grover`や`Ferrum`といった gem があります。

`Ferrum`は Ruby で Chrome ブラウザを操作できる API を提供してくれて、`Grover`のように Puppeteer を経由する必要がありません。

今回は`Ferrum`で PDF 生成してみます。

# 前提

- Rails 7.1.2
  - Rails 7.1 から`$ rails new`で Dockerfile が生成されるようになっています
  - `$ rails g scaffold post title:string body:text published:boolean`実行済み

# 必要なのは Chrome or Chromium だけ

Dockerfile で`chromium`をインストールしておく

```diff:Dockerfile
# Install packages needed for deployment
RUN apt-get update -qq && \
-　　　　 apt-get install --no-install-recommends -y curl libvips postgresql-client && \
+   apt-get install --no-install-recommends -y curl libvips postgresql-client chromium && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives
```

# Ferrum で PDF 生成

```ruby:posts_controller.rb
class PostsController < ApplicationController
  before_action :set_post, only: %i[ show edit update destroy download_pdf ]

  # 中略

  def download_pdf
    html = render_to_string(template: 'posts/_post', layout: 'pdf', locals: { post: @post })
    pdf = html2pdf(html)
    send_data pdf, filename: 'post.pdf', type: 'application/pdf'
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_post
      @post = Post.find(params[:id])
    end

　　　　　　　　# Ferrumを使ってHTMLからPDFを生成
    def html2pdf(html)
      # chromiumへのパスとno-sandbox browserオプションを渡す
      browser = Ferrum::Browser.new(browser_path: '/usr/bin/chromium', browser_options: { 'no-sandbox': nil })

      header_html = render_to_string('pdf/header', layout: false)
      footer_html = render_to_string('pdf/footer', layout: false)

      browser.goto("data:text/html,#{html}")

      pdf = browser.pdf(
        format: :A4,
        encoding: :binary,
        display_header_footer: true,
        header_template: header_html,
        footer_template: footer_html,
      )

      browser.quit

      pdf
    end
end

```

:::details その他のコード

- download_pdf アクションのルート追加

```ruby:cofig/routes.rb
Rails.application.routes.draw do
  resources :posts do
    member do
      get :download_pdf
    end
  end
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  # Reveal health status on /up that returns 200 if the app boots with no exceptions, otherwise 500.
  # Can be used by load balancers and uptime monitors to verify that the app is live.
  get "up" => "rails/health#show", as: :rails_health_check

  # Defines the root path route ("/")
  # root "posts#index"
end
```

- PDF ダウンロードボタンを追加

```diff:posts/show.html.erb
<p style="color: green"><%= notice %></p>

<%= render @post %>

<div>
  <%= link_to "Edit this post", edit_post_path(@post) %> |
  <%= link_to "Back to posts", posts_path %>

  <%= button_to "Destroy this post", @post, method: :delete %>
+ <%= link_to 'Download PDF', download_pdf_post_path(@post) %>
</div>
```

- PDF 用レイアウト

```erb:layouts/pdf.html.erb
<!DOCTYPE html>
<html>
  <head>
    <title>PDF generated by Ferrum</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
  </head>

  <body>
    <%= yield %>
  </body>
</html>
```

```erb:layouts/pdf/header.html.erb
<div style="margin-left: 0.5cm; font-size: 9px; width: 100%;">
  <span class="date"></span>
</div>
```

```erb:layouts/pdf/footer.html.erb
<div style="position: relative; border-top: 1px solid black; margin: 0.5cm; font-size: 9px; width: 100%;">
  <div style="position: absolute; width: 100%; top: 0.2cm; text-align: center;">
    <span class='pageNumber'></span> / <span class='totalPages'></span>
  </div>
  <div style="position: absolute; right: 0; top: 0.2cm;">generated by Ferrum</div>
</div>
```

:::

リポジトリはこちら

https://github.com/kmkkiii/generate_pdf_by_ferrum

## フォーマット指定

標準的な用紙サイズがオプションとして用意されています。
`format`オプションで用紙サイズを指定するか、`paper_width`と`paper_height`を指定します。

```
standard paper sizes :letter, :legal, :tabloid, :ledger, :A0, :A1, :A2, :A3, :A4, :A5, :A6
```

https://github.com/rubycdp/ferrum?tab=readme-ov-file#pdfoptions--string--boolean

## Docker で利用する場合

Docker コンテナ内で、root ユーザーとして実行する場合は以下のオプションを渡す必要があります。

```ruby
Ferrum::Browser.new(browser_options: { 'no-sandbox': nil })
```

また、M1 Mac 上の Docker コンテナ内で Ferrum を実行する際に Chrome プロセスがクラッシュするという報告があるみたいです。今のところ自分の環境では発生していませんが不穏ですね。

## ヘッダーとフッターはカスタマイズできる

ヘッダーとフッターは HTML 文字列を渡してカスタマイズできます。

:::message alert
`display_header_footer： true`にしても表示されないと思っていましたが、極小文字サイズで見切れてしまっていただけでした...。
font-size だけは少なくとも style 設定する必要がありますね。
:::

`display_header_footer: true`にするとヘッダーとフッター両方表示してしまうので、どちらかだけ表示したい場合は空文字を渡します。

https://chromedevtools.github.io/devtools-protocol/tot/Page/#method-printToPDF

# 最後に

ここまで読んでくださり、ありがとうございました！
帳票や請求書などの PDF 作成機能実装時は`wkhtmltopdf`に大変お世話になったので悲しいです。
お世話になっているライブラリに対して、個人で少しでも貢献できるようになりたいと強く感じます。

# 参考

https://blog.willnet.in/entry/2023/02/10/233053

https://tech.smarthr.jp/entry/2023/07/03/170509
