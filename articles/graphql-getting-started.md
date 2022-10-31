---
title: "GraphQLに入門しました" # 記事のタイトル
emoji: "🐢" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["GraphQL", "Rails", "React", "Apollo", "Typescript"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# はじめに

研修として Ruby on Rails × GraphQL × React × TypeScript を使ったタスク管理アプリを作っており、その過程で調べて実装した内容となります。少しでも参考になれば幸いです。
GraphQL に入門したてなので、間違っている点やこうした方がいいよという点がありましたらコメントでご指摘頂けると大変ありがたいです。

# 導入

## バックエンド(Rails)

graphql を使うための gem をインストールします。

```ruby:Gemfile
gem 'graphql'
```

```
$ bundle install
```

以下のコマンドで必要なファイルを自動生成します。
ブラウザで GraphQL の実行確認ができる graphiql-rails も併せて追加されるので、もう一度`bundle install`が必要になるかもしれません。

```
$ rails g graphql:install
```

生成されたファイルはデフォルトでは`app/graphql/types`以下に配置されますが、
ファイルを追加していくと対象ファイルを探すのが辛くなってきます。
そのため、自前でフォルダを作成して整理すると良い感じです。
フォルダ構成はいろいろな記事で紹介されていますのでお好みでどうぞ。

```
ここに作成したフォルダ構成をtreeで書く
```

GraphiQL にアクセスするためのルーティングも追加されているはずのなので、
サーバー起動後、ブラウザで`http://localhost:3000/graphiql`にアクセスすると IDE が開き、GraphQL のクエリを実行して確認することができます。

```ruby:config/routes.rb
Rails.application.routes.draw do

  if Rails.env.development?
    mount GraphiQL::Rails::Engine, at: "/graphiql", graphql_path: "/graphql"
  end
  post "/graphql", to: "graphql#execute"

end
```

## フロントエンド(React、TypeScript)

```
$ yarn add
```

ApolloClient
GraphQL Code Generator

TODO：スキーマ指定からエンドポイント指定に変えて試してみる。

# Query 実装編

## 全件取得

## id を使って 1 件取得

## 複数テーブルから取得

## ページネーション

# Mutation 実装編

## 登録(Create)

## 更新(Update)

## 削除(Delete)

## 番外編 認証情報を扱いたいときには

研修では認証機能を `devise` と `devise_token_auth` の gem を用いて実装しました。
ログイン中のユーザーに紐づいたデータのみを取り扱いたい場合に、devise のヘルパーメソッドである current_user を使用したいケースがあるかと思います。
そんな時には GraphqlController に少し手を加えるだけで、Query や Mutation のリソルバでも current_user が使用できるようになります。

```ruby:graphql_controller.rb

```

# 参考

-
