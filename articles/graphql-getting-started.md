---
title: "GraphQLに入門しました" # 記事のタイトル
emoji: "🐢" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["GraphQL", "Rails", "React", "Apollo", "Typescript"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# はじめに

研修としてRuby on Rails × GraphQL × React × TypeScriptを使ったタスク管理アプリを作っており、その過程で調べて実装した内容となります。少しでも参考になれば幸いです。
GraphQLに入門したてなので、間違っている点やこうした方がいいよという点がありましたらコメントでご指摘頂けると大変ありがたいです。

# 導入

## バックエンド(Rails)

graphqlを使うためのgemをインストールします。

```
# Gemfile
gem 'graphql'
```

```
$ bundle install
```

```
$ rails g graphql:install 
```

ブラウザでGraphQLの実行確認ができるgraphiql-railsも自動で追加されるのでもう一度`bundle install`が必要になるかもしれません。

生成されたファイルはデフォルトではapp/graphql/types以下に配置されるので、ファイルを追加していくと対象ファイルを探すのが辛くなってきます。
そのため、フォルダを作成して整理すると良いです。



## フロントエンド(React、TypeScript)

ApolloClient
GraphQL Code Generator

TODO：スキーマ指定からエンドポイント指定に変えて試してみる。


# Query実装編

## 全件取得




## idを使って1件取得


## 複数テーブルから取得


## ページネーション


# Mutation実装編

## 登録(Create)


## 更新(Update)


## 削除(Delete)


# 参考

- 