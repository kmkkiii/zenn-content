---
title: "Tanstack FormでNext.jsのServer Actionsを使ってみた" # 記事のタイトル
emoji: "😜" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Nextjs", "ServerActions", "TanstackForm"] # タグ。["markdown", "rust", "aws"]のように指定する
publication_name: "mofmof_inc"
published: false # 公開設定（falseにすると下書き）
---

## はじめに

Next.js v14 から Stable になった Server Actions。
それに対応した Form ライブラリとして Tanstack Form を試してみます。

Tanstack Form は現時点で v0 の β 版ですが、以下の特徴を備えた Form ライブラリです。

依存 0 でさまざまな機能セットを提供しています。

- フレームワークにとらわれないデザイン
- ファーストクラスの TypeScript サポート
- ヘッドレス
- タイニー/ゼロデプス
- きめ細かなリアクティブ・コンポーネント/フック
- 拡張性とプラグインアーキテクチャ
- モジュラーアーキテクチャ
- フォーム/フィールドバリデーション
- 非同期バリデーション
- 組み込みの非同期バリデーションデバウンス
- 設定可能な検証イベント
- 深くネストされたオブジェクト/配列フィールド

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

## Form 実装

名前とメールアドレス、住所の入力 Form を例として実装してみました。

```ts

```

# 最後に

ここまで読んでくださり、ありがとうございました！
