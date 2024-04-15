---
title: "Tanstack FormでNext.jsのServer Actionsを使ってみた" # 記事のタイトル
emoji: "😜" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["nextjs", "serveractions", "tanstackform"] # タグ。["markdown", "rust", "aws"]のように指定する
publication_name: "mofmof_inc"
published: true # 公開設定（falseにすると下書き）
---

## はじめに

Next.js v14 から Stable になった Server Actions。
それに対応した フォームライブラリとして Tanstack Form を試してみます。

Tanstack Form は現時点で v0 の β 版ですが、以下の特徴を備えた フォームライブラリです。

- ファーストクラスの TypeScript サポート
- シンプルで簡潔な API
- クライアントサイドとサーバーサイド両方のバリデーションに対応
- ヘッドレス
  - 任意の UI フレームと組み合わせて実装可能
- アグノスティックな設計
  - React、Vue、Angular, Solid, Lit と複数のフレームワークに対応
  - フレームワーク間で再利用可能
- 非同期バリデーションとデバウンシングを組み込みでサポート

https://tanstack.com/form/latest

## 準備

https://tanstack.com/form/latest/docs/framework/react/guides/ssr

### create-next-app

```sh
 % npx create-next-app@latest
Need to install the following packages:
create-next-app@14.2.1
Ok to proceed? (y) y
✔ What is your project named? … nextjs_tanstack_form_example
✔ Would you like to use TypeScript? … No / Yes
✔ Would you like to use ESLint? … No / Yes
✔ Would you like to use Tailwind CSS? … No / Yes
✔ Would you like to use `src/` directory? … No / Yes
✔ Would you like to use App Router? (recommended) … No / Yes
✔ Would you like to customize the default import alias (@/*)? … No / Yes
Creating a new Next.js app in /Users/kmkkiii/Development/nextjs_tanstack_form_example.

Using npm.

Initializing project with template: app-tw


Installing dependencies:
- react
- react-dom
- next

Installing devDependencies:
- typescript
- @types/node
- @types/react
- @types/react-dom
- postcss
- tailwindcss
- eslint
- eslint-config-next


added 360 packages, and audited 361 packages in 18s

133 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
Initialized a git repository.

Success! Created nextjs_tanstack_form_example at /Users/kmkkiii/Development/nextjs_tanstack_form_example
```

### tanstack-form インストール

```sh
% npm i @tanstack/react-form
```

今回 validator には zod を使用します。
他にも yup、valibot がサポートされていますのでお好みでインストールしてください。

```sh
% npm i @tanstack/zod-form-adapter zod
```

## フォーム実装

名前とメールアドレスの入力フォームを例として実装してみました。

![](/images/nextjs_server_actions_tanstack_form/form.png)

### zod のスキーマを定義

```ts:schema.ts
import { z } from "zod";

export const schema = z.object({
  name: z.string().min(1, "Client validation: 名前を入力してください"),
  email: z
    .string()
    .min(1, "Client validation: メールアドレスを入力してください")
    .email("Client validation: メールアドレスの形式が正しくありません"),
});
```

### FormFactory オブジェクトを作成

useForm フックを使用する方法もありますが、createFormFactory を使用するとフォームの作成・管理に便利なユーティリティ関数を含むオブジェクトを作成してくれて再利用が可能となります。
また、サーバーサイドで実行されるバリデーションを設定するための`onServerValidate`プロパティを指定できるようになっています。

```ts:exampleFormFactory.ts
import { createFormFactory } from "@tanstack/react-form";
import { zodValidator } from "@tanstack/zod-form-adapter";

export const exampleFormFactory = createFormFactory({
  validatorAdapter: zodValidator,
  defaultValues: {
    name: "",
    email: "",
  },
  onServerValidate({ value }) {
    let errors = [];

    if (value.name == "test") {
      errors.push("Server validation: すでに登録されている名前です");
    }
    if (value.email == "test@example.com") {
      errors.push("Server validation: すでに登録されているメールアドレスです");
    }

    return errors.join(", ");
  },
});
```

### Server Action を定義

```ts:validateAction.ts
"use server";

import { exampleFormFactory } from "@/app/_components/exampleFormFactory";

export default async function validateAction(
  prev: unknown,
  formData: FormData
) {
  return await exampleFormFactory.validateFormData(formData);
}
```

### フォームコンポーネントを作成

- `useFormState`へ定義した Server Action とフォームの初期状態を渡します。
  - 返された 値(action) を form の action 属性 に設定することで送信時にトリガーされ、サーバーでのみ実行されます。
- `useForm` の各プロパティについて
  - `transform` では、`useTransform`でフォームの状態と`useFormState`に返された Server Action の実行結果(state)をマージしています。
  - `validators` では、onChange イベントに対して zod スキーマによるバリデーションを設定しています。
  - `onSubmit`では、有効なフォームデータが送信された場合に呼び出される処理を設定します。
  - `onSubmitInvalid`では、無効なフォームデータが送信された場合に呼び出される処理を設定します。

```ts:ExampleForm.tsx
"use client";

import { exampleFormFactory } from "@/app/_components/exampleFormFactory";
import { FormApi, mergeForm, useTransform } from "@tanstack/react-form";
import { useFormState } from "react-dom";
import validateAction from "../_actions/validateAction";
import { schema } from "@/lib/zod/schema";

const ExampleForm = () => {
  const [state, action] = useFormState(
    validateAction,
    exampleFormFactory.initialFormState
  );

  const { useStore, Subscribe, Field, handleSubmit } =
    exampleFormFactory.useForm({
      transform: useTransform(
        (baseForm: FormApi<any, any>) => mergeForm(baseForm, state),
        [state]
      ),
      validators: { onChange: schema },
      onSubmit: async ({ value, formApi }) => {
        console.log(value, formApi);
      },
      onSubmitInvalid: ({ value, formApi }) => {
        console.error(value, formApi);
      },
    });

  const formErrors = useStore((formState) => formState.errors);

  return (
    <>
      <form action={action} onSubmit={() => handleSubmit()}>
        {formErrors.map((error) => (
          <p className="text-red-600" key={error as string}>
            {error}
          </p>
        ))}

        <Field name="name">
          {(field) => (
            <div className="flex flex-col">
              <label htmlFor="email">名前</label>
              <input
                className="text-black border border-gray p-2 mb-4 w-1/3"
                name="name"
                type="text"
                value={field.state.value}
                onChange={(e) => field.handleChange(e.target.value)}
              />
            </div>
          )}
        </Field>
        <Field name="email">
          {(field) => (
            <div className="flex flex-col">
              <label htmlFor="email">メールアドレス</label>
              <input
                className="text-black border border-gray p-2 mb-4 w-1/3"
                name="email"
                type="email"
                value={field.state.value}
                onChange={(e) => field.handleChange(e.target.value)}
              />
            </div>
          )}
        </Field>
        <Subscribe
          selector={(formState) => [
            formState.canSubmit,
            formState.isSubmitting,
          ]}
        >
          {([canSubmit, isSubmitting]) => (
            <button
              type="submit"
              disabled={!canSubmit}
              className="text-white bg-blue-700 hover:bg-blue-800 px-4 py-2 mt-2 rounded-lg disabled:opacity-50 disabled:cursor-not-allowed"
            >
              {isSubmitting ? "..." : "送信"}
            </button>
          )}
        </Subscribe>
      </form>
    </>
  );
};
export default ExampleForm;
```

### ページでフォームコンポーネントを使用する

```ts:page.tsx
import ExampleForm from "./_components/ExampleForm";

export default function Home() {
  return (
    <main className="min-h-screen p-24">
      <ExampleForm />
    </main>
  );
}
```

## 動作確認

### クライアントサイドのバリデーション

入力や送信によってトリガーされ、zod スキーマで定義したバリデーションが機能します。

![](/images/nextjs_server_actions_tanstack_form/client-side_validation.png)

### サーバーサイドのバリデーション

名前に`test`、メールアドレスに`test@example.com`と入力して送信するとサーバーサイドのバリデーションが機能していることが確認できます。

![](/images/nextjs_server_actions_tanstack_form/server-side_validation.png)

## Tanstack Form を使ってみた感想

確かに RHF の幅広い API と比べると、シンプルで迷わなくて良さそうでした。
また、依存が 0 で他のフレームワークにも流用できる設計という点も開発規模によっては嬉しい点でしょうか。
v1 がリリースされる頃には、さらに良いライブラリになっているのではないかと期待が高まりますね。

実装していて気になったのは、フィールド単位ではなくフォームレベルでバリデーションを設定した場合に、どのフィールドがエラーになったのかを知る方法が現状ないことでした。
この問題については、現在以下の PR で開発が進められているようなので、近いうちに解消されると思われます！

https://github.com/TanStack/form/pull/656

# 最後に

ここまで読んでくださり、ありがとうございました！
