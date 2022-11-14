---
title: "GraphQLに入門しました" # 記事のタイトル
emoji: "🐢" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["GraphQL", "Rails", "React", "Apollo", "Typescript"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# はじめに

10 月から株式会社 mofmof に入社し、研修として Ruby on Rails × GraphQL × React × TypeScript を使ったタスク管理アプリを作ったので、その実装内容を抜粋してまとめてみました。
これから GraphQL を使っていこうとしている方のとっかかりとして、少しでも参考になれば幸いです。
間違っている点やこうした方がいいよという点がありましたらコメントでご指摘頂けると大変ありがたいです。

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

実行後、`app/graphql/types`と`app/graphql/mutations`のフォルダが作成されますが、
ファイルを追加していくとこれらのフォルダから対象ファイルを探すのが辛くなってきます。
そのため、自前でフォルダを作成して整理すると良い感じです。今回は以下のような構成にしました。
フォルダ構成はいろいろな記事で紹介されていますのでお好みでどうぞ。

```
app/graphql
├── input_types <- 追加
├── mutations
├── object_types <- 追加
├── queries <- 追加
└── types
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
$ yarn add graphql @apollo/client
$ yarn add -D typescript @graphql-codegen
```

ApolloClient

GraphQL Code Generator

TODO：スキーマ指定からエンドポイント指定に変えて試してみる。

# Query 実装編

Query のリソルバ作成

Object の Type 生成

QueryType に追加

スキーマダンプ
以下のコマンドを実行すると、ルートに`schema.graphql`と`schema.json`が生成されます。

```
$ rails graphql:schema:dump
```

ts でクエリをかく

yarn codegen (GraphQL Code Generator)で型と Hooks 生成

```
$ yarn codegen
```

生成された Hooks をコンポーネントに導入
生成された`graphql.ts`には「use〜Query」or「use〜Mutation」という名前で Hooks が生成されます。
基本的な使い方はコメントで例が示されています。

## 全件取得

## id を使って 1 件取得

## 複数テーブルから取得

## ページネーション

# Mutation 実装編

## 登録(Create)

## 更新(Update)

## 削除(Delete)

## 【番外編その１】 認証情報を扱いたい

研修では認証機能を `devise` と `devise_token_auth` の gem を用いて実装しました。
ログイン中のユーザーに紐づいたデータのみを取り扱いたい場合に、devise のヘルパーメソッドである current_user を使用したいケースがあるかと思います。
そんな時には下記のように GraphqlController へ少し手を加えることによって、`context[:current_user]`という形で current_user を参照できるようになります。

```diff ruby:graphql_controller.rb
class GraphqlController < ApplicationController

  def execute
    variables = prepare_variables(params[:variables])
    query = params[:query]
    operation_name = params[:operationName]
    context = {
      # Query context goes here, for example:
+     current_user: current_user,　# コメントアウトを外す
-     #current_user: current_user,
    }
    result = MyappSchema.execute(query, variables:, context:, operation_name:)
    render json: result
  rescue StandardError => e
    raise e unless Rails.env.development?

    handle_error_in_development(e)
  end

  # 中略
end

```

## 【番外編その２】 引数が不要な Mutation を定義したい

フロント側で`input: {}`のように引数を書くか、GraphQL::Schema::Mutation を継承して InputObject を自動生成しないようにする方法が考えられます。

# 参考 3

-
