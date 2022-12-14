---
title: "Rails×ApolloClient×React×TypeScriptでGraphQLに入門しました" # 記事のタイトル
emoji: "🐢" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["GraphQL", "Rails", "React", "Apollo", "Typescript"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに

この記事は、[mofmof Advent Calendar 2022](https://qiita.com/advent-calendar/2022/mofmof) **15 日目**の記事です。
10 月から[株式会社 mofmof](https://www.mof-mof.co.jp/) に入社し、研修として Ruby on Rails × GraphQL × React × TypeScript を使ったタスク管理アプリを作りました。その中から GraphQL に関する実装内容を抜粋してまとめてみました。
これから GraphQL を使っていこうとしている方のとっかかりとして、少しでも参考になれば幸いです。

# 導入

## サーバーサイド(Rails)

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

## クライアントサイド(React、TypeScript)

クライアント側で GraphQL を扱いやすくするために以下のライブラリを追加します。
設定等は割愛しますので、それぞれ Get Started のドキュメントをご覧ください。

```
$ yarn add graphql @apollo/client
$ yarn add -D typescript @graphql-codegen
```

https://www.apollographql.com/docs/react/get-started

https://the-guild.dev/graphql/codegen/docs/getting-started

# Query 実装編

サーバーサイドからクライアントサイドの順で以下の流れで実装していきました。

1. Object の Type 定義

2. Query のリソルバ定義

3. QueryType にリゾルバを追加

4. スキーマダンプ
   以下のコマンドを実行すると、プロジェクトルートに`schema.graphql`と`schema.json`が生成されます。

```
$ rails graphql:schema:dump
```

生成されたスキーマは GraphQL Code Generator から参照するために`codegen.yml`の `schema`プロパティにパスを指定します。

```yaml:codegen.yml
overwrite: true
schema: "./schema.graphql"
documents: "./app/javascript/graphql/**/*.ts"
generates:
  ./app/javascript/generated/graphql.ts:
    plugins:
      - "typescript"
      - "typescript-operations"
      - "typescript-react-apollo"
    config:
      - withHOC: false
      - withComponent: false
      - withHooks: true
hooks:
  afterOneFileWrite:
    - prettier --write
```

5. ts でクエリをかく

6. GraphQL Code Generator で型と Hooks 生成

```
$ yarn codegen
```

7. 生成された Hooks をコンポーネントに導入
   生成された`graphql.ts`には「use〜Query」or「use〜Mutation」という名前で Hooks が生成されます。基本的な使い方はコメントで例が示されています。

```typescript
/**
 * __useFetchTaskByIdQuery__
 *
 * To run a query within a React component, call `useFetchTaskByIdQuery` and pass it any options that fit your needs.
 * When your component renders, `useFetchTaskByIdQuery` returns an object from Apollo Client that contains loading, error, and data properties
 * you can use to render your UI.
 *
 * @param baseOptions options that will be passed into the query, supported options are listed on: https://www.apollographql.com/docs/react/api/react-hooks/#options;
 *
 * @example
 * const { data, loading, error } = useFetchTaskByIdQuery({
 *   variables: {
 *      id: // value for 'id'
 *   },
 * });
 */
export function useFetchTaskByIdQuery(
  baseOptions: Apollo.QueryHookOptions<
    FetchTaskByIdQuery,
    FetchTaskByIdQueryVariables
  >
) {
  // 中略
}
```

ここからは実装したリゾルバと TypeScript で定義したクエリを列挙していきます。

## 全件取得

:::details 実装例

```ruby:app/graphql/queries/tasks.rb
module Queries
  class Tasks < Queries::BaseQuery
    type [ObjectTypes::TaskType], null: false

    def resolve
      ::Task.all
    end
  end
end
```

```typescript:app/javascript/graphql/queries/tasks.ts
import { gql } from "@apollo/client";

export const FETCH_TASKS = gql`
  query FetchTasks {
    tasks {
      id
      title
      detail
      limitOn
    }
  }
`;
```

:::

## id を使って 1 件取得

:::details 実装例

```ruby:app/graphql/queries/task.rb
module Queries
  class Task < Queries::BaseQuery
    type ObjectTypes::TaskType, null: false
    argument :id, ID, required: true

    def resolve(id:)
      ::Task.find(id)
    end
  end
end
```

```typescript:app/javascript/graphql/queries/task.ts
import { gql } from "@apollo/client";

export const FETCH_TASK_BY_ID = gql`
  query FetchTaskById($id: ID!) {
    task(id: $id) {
      id
      title
      detail
      limitOn
    }
  }
`;
```

:::

## 複数テーブルから取得

モデル同士のリレーションを設定して ObjectTypes にフィールドを追加することで取得できます。
以下の実装では、Task と Status が 1 対多の関係になっています。

:::details 実装例

```diff ruby:app/graphql/object_types/task_type.rb
# frozen_string_literal: true

module ObjectTypes
  class TaskType < Types::BaseObject
    field :id, ID, null: false
    field :title, String, null: false
    field :detail, String
    field :limit_on, GraphQL::Types::ISO8601Date, null: false
+   field :status_id, ID
+   field :status, ObjectTypes::StatusType
    field :created_at, GraphQL::Types::ISO8601DateTime, null: false
    field :updated_at, GraphQL::Types::ISO8601DateTime, null: false
  end
end
```

```typescript:app/javascript/graphql/queries/tasks.ts
import { gql } from "@apollo/client";

export const FETCH_TASKS = gql`
  query FetchTasks {
    tasks {
      id
      title
      detail
      limitOn
      statusId
      status {
        id
        name
      }
    }
  }
`;
```

:::

## ページネーション

クエリ定義の Type に対して`connection_type`を付けることで、
クエリの引数で件数を指定したり、pageInfo で次ページの有無やカーソル文字列等のメタデータを取得したりできるようになります。

:::details 実装例

```ruby:app/graphql/queries/tasks.rb
module Queries
  class Tasks < Queries::AuthRequiredQuery
    type ObjectTypes::TaskType.connection_type, null: false

    def resolve
      ::Task.all
    end
  end
end
```

```typescript:app/javascript/graphql/queries/tasks.ts
import { gql } from "@apollo/client";

export const FETCH_TASKS = gql`
  query FetchTasks(
    $first: Int
    $last: Int
    $before: String
    $after: String
  ) {
    tasks(
      first: $first
      last: $last
      before: $before
      after: $after
    ) {
      edges {
        node {
          id
          title
          detail
          limitOn
          status {
            id
            name
          }
        }
      }
      pageInfo {
        hasPreviousPage
        hasNextPage
        startCursor
        endCursor
      }
    }
  }
`;
```

:::

# Mutation 実装編

パラメータを一つ一つリゾルバに定義するのは大変なので、`input_types`にまとめて定義しておきます。

```ruby:app/graphql/input_types/task.rb
module InputTypes
  class Task < Types::BaseInputObject
    graphql_name 'TaskInput'

    argument :title, String, required: true
    argument :detail, String, required: false
    argument :limit_on, String, required: true
    argument :status_id, ID, required: true
  end
end
```

## 登録(Create)

:::details 実装例

```ruby:app/graphql/mutations/create_task.rb
module Mutations
  class CreateTask < Mutations::BaseMutation
    field :task, ObjectTypes::TaskType, null: false

    argument :params, InputTypes::Task, required: true

    def resolve(params:)
      task = ::Task.create!(params.to_h)
      { task: }
    rescue StandardError => e
      GraphQL::ExecutionError.new(e.message)
    end
  end
end

```

```typescript:app/javascript/graphql/mutations/createTask.ts
import { gql } from "@apollo/client";

export const CREATE_TASK = gql`
  mutation CreateTask($params: TaskInput!) {
    createTask(input: { params: $params }) {
      task {
        id
        title
        detail
        limitOn
        statusId
      }
    }
  }
`;
```

:::

## 更新(Update)

:::details 実装例

```ruby:app/graphql/mutations/update_task.rb
module Mutations
  class UpdateTask < BaseMutation
    field :task, ObjectTypes::TaskType, null: false

    argument :id, ID, required: true
    argument :params, InputTypes::Task, required: true

    def resolve(id:, params:)
      task = ::Task.find(id)
      task.update!(params.to_h)
      { task: }
    rescue StandardError => e
      GraphQL::ExecutionError.new(e.message)
    end
  end
end
```

```typescript:app/javascript/graphql/mutations/updateTask.ts
import { gql } from "@apollo/client";

export const UPDATE_TASK = gql`
  mutation UpdateTask($id: ID!, $params: TaskInput!) {
    updateTask(input: { id: $id, params: $params }) {
      task {
        id
        title
        detail
        limitOn
        statusId
      }
    }
  }
`;
```

:::

## 削除(Delete)

:::details 実装例

```ruby:app/graphql/mutations/delete_task.rb
module Mutations
  class DeleteTask < BaseMutation
    field :id, ID, null: false

    argument :id, ID, required: true

    def resolve(id:)
      ::Task.find(id).destroy!
      { id: }
    rescue StandardError => e
      GraphQL::ExecutionError.new(e.message)
    end
  end
end
```

```typescript:app/javascript/graphql/mutations/deleteTask.ts
import { gql } from "@apollo/client";

export const DELETE_TASK = gql`
  mutation DeleteTask($id: ID!) {
    deleteTask(input: { id: $id }) {
      id
    }
  }
`;
```

:::

# 【番外編その１】 リゾルバで current_user を使いたい

研修では認証機能を `devise` と `devise_token_auth` の gem を用いて実装しました。
ログイン中のユーザーに紐づいたデータのみを取り扱いたい場合に、devise のヘルパーメソッドである current_user を使用したいケースがあるかと思います。
そんな時には下記のように GraphqlController へ少し手を加えることによって、`context[:current_user]`という形で current_user を参照できるようになります。

:::details 実装例

```diff ruby:graphql_controller.rb
class GraphqlController < ApplicationController

  def execute
    variables = prepare_variables(params[:variables])
    query = params[:query]
    operation_name = params[:operationName]
    context = {
      # Query context goes here, for example:
-     #current_user: current_user,
+     current_user: current_user,　# コメントアウトを外す
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

今回はトークン認証を行なっているため、ApolloClient で以下の設定を追加することにより、GraphqlController 経由でログイン中のユーザー情報が扱えるようにしています。

```typescript:application.tsx
import * as React from "react";
import { createRoot } from "react-dom/client";
import { ApolloClient, InMemoryCache, ApolloProvider } from "@apollo/client";
import { setContext } from "@apollo/client/link/context";
import App from "../App";
import Cookies from "js-cookie";
import { createUploadLink } from "apollo-upload-client";

const container = document.getElementById("root") as HTMLElement;
const root = createRoot(container);

const authLink = setContext((_, { headers }) => {
  return {
    headers: {
      "access-token": Cookies.get("_access_token"),
      client: Cookies.get("_client"),
      uid: Cookies.get("_uid"),
    },
  };
});

const client = new ApolloClient({
  link: authLink.concat(
    createUploadLink({
      uri: "/graphql",
    })
  ),
});

document.addEventListener("DOMContentLoaded", () => {
  root.render(
    <React.StrictMode>
      <ApolloProvider client={client}>
        <App />
      </ApolloProvider>
    </React.StrictMode>
  );
});

```

:::

# 【番外編その２】 引数を取らない Mutation を定義したい

フロント側で`input: {}`のように引数を書くか、GraphQL::Schema::Mutation を継承して InputObject を自動生成しないようにする方法が考えられます。

https://qiita.com/ham0215/items/c11324bfc98e56778891

# 最後に

研修を通して GraphQL と少し仲良くなれた気がします。
今後は[Production Ready GraphQL](https://book.productionreadygraphql.com/)を読んでベストプラクティスを学んでいきたいです。
ここまで読んで下さってありがとうございました！
