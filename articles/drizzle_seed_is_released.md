---
title: 'Drizzle ORMのdrizzle-seed packageがリリースされました🎉' # 記事のタイトル
emoji: '🌱' # アイキャッチとして使われる絵文字（1文字だけ）
type: 'tech' # tech: 技術記事 / idea: アイデア記事
topics: ['DrizzleORM', 'seed'] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
publication_name: 'mofmof_inc'
---

:::message
この記事は「[mofmof Advent Calendar 2024](https://qiita.com/advent-calendar/2024/mofmof)」1 日目の記事です。
:::

## New Package: drizzle-seed

Drizzle ORM に待望の drizzle-seed パッケージがリリースされました(2024-11-22)。

https://github.com/drizzle-team/drizzle-orm/releases/tag/0.36.4

ドキュメントはこちら

https://orm.drizzle.team/docs/seed-overview

これまでは migration ファイルを作成して INSERT 文を書いていました。

https://orm.drizzle.team/docs/kit-custom-migrations

## 特徴

### 決定論的データ生成(Deterministic Data Generation)

決定論的データ生成とは、同じ seed 値を使って生成したデータは毎回同じになるということです。
マイクラのワールド生成の仕組みと同じですね。

### 擬似乱数生成(Pseudorandom Number Generator (pRNG))

数値シーケンスの生成が使用されますが、前述の seed 値によってランダム性が制御されるため、再現が可能です。
ドキュメントでは以下の 3 つのメリットが示されています。

> pRNG を使用する利点:
>
> - 一貫性: テストが毎回同じデータで実行されるようにします。
> - デバッグ: 一貫したデータ セットを提供することで、バグの再現と修正が容易になります。
> - コラボレーション: チーム メンバーはシード番号を共有して同じデータ セットで作業できます。
>   https://orm.drizzle.team/docs/seed-overview#benefits-of-using-a-prng
>   (Google 翻訳使用)

## 使ってみた

example プロジェクトはこちらです。今回は DB に PostgreSQL を使用しています。
https://github.com/kmkkiii/nextjs-drizzle-seed-example

drizzle のスキーマ作成や DB への反映(`drizzle-kit push`)は済んでいる前提でインストールから！

```sh
npm i drizzle-seed
```

### 使い方

基本的には DB インスタンスとスキーマを seed 関数に渡すだけ。
デフォルトでは 10 件のデータが生成されます。

```ts: db/seeds.ts
import { seed } from 'drizzle-seed';
import * as schema from './schema';
import dotenv from 'dotenv';
import { drizzle } from 'drizzle-orm/node-postgres';

dotenv.config();

async function main() {
  const db = drizzle(process.env.DATABASE_URL!);
  await seed(db, schema);
}

main();
```

```sh
// tsxで実行
> npx tsx db/seeds.ts
```

### オプション

`seed` 関数には 2 つのオプションが指定できます。

#### count

生成するデータ件数を指定するオプションです。

```ts
await seed(db, schema, { count: 1000 });
```

#### seed

seed 値を指定することで毎回同じデータを生成する＝再現が可能です。

```ts
await seed(db, schema, { seed: 12345 });
```

### リセット

DB のリセットは `reset` 関数を使用します。
`seed` 関数と同じように DB インスタンスとスキーマを渡すだけ！

```ts :db/reset.ts
import * as schema from './schema';
import { reset } from 'drizzle-seed';
import dotenv from 'dotenv';
import { drizzle } from 'drizzle-orm/node-postgres';

dotenv.config();

async function main() {
  const db = drizzle(process.env.DATABASE_URL!);
  await reset(db, schema);
}

main();
```

```sh
// tsxで実行
> npx tsx db/reset.ts
```

### 改良(Refinements)

`refine`関数を使用して seed の挙動を改良することも可能です。

スキーマで定義したテーブルの変数名をキーとしたオブジェクトに対して、drizzle-seed が提供するジェネレーター関数を使用して改良します。

明示的に改良しなくても、スキーマで定義したカラムの型や名前から推論してくれるのか適切なジェネレーター関数を使用してくれたりします。

`with`は親テーブルに関連するエンティティを生成します。親テーブルごとに生成する数も指定できます。

```ts
await seed(db, schema).refine((f) => ({
  users: {
    columns: {
      id: f.int({
        minValue: 1,
        isUnique: true,
      }),
      name: f.fullName(),
      // emailは設定しなくてもf.email()を使ってメールアドレスを生成してくれる
    },
    count: 10,
    with: {
      posts: 10,
    },
  },
}));
```

### ジェネレーター関数

https://orm.drizzle.team/docs/seed-functions

`int`や`boolean`, `string`といったプリミティブ型？はもちろん、以下のようなよく使いそうなデータのジェネレーター関数が揃っています！

- タイムスタンプ
  - `date`
  - `time`
  - `timestamp`
  - `datetime`
  - `year`
- 名前
  - `firstName`
  - `lastName`
  - `fullName`
- 住所
  - `country`
  - `state`
  - `postCode`
  - `city`
  - `streetAddress`
- 連絡先
  - `email`
  - `phoneNumber`
- 文章
  - `loremIpsum`
    - 生成する文章数を指定できる
- 2D データ
  - `point`
  - `line`

### 加重ランダム(Weighted Random)

weighted random API を使用することで生成されるデータを確率的に制御することができます。
例えば example プロジェクトでは、ユーザーの年齢層の割合を weight で設定しています。

https://github.com/kmkkiii/nextjs-drizzle-seed-example/blob/2e6ad1391ece233be9f78f590a58bac793d1c40b/db/seeds.ts#L5-L34

## 現時点では制限されていること

- テーブル間の参照を適切に推論できない
  - with オプションではスキーマに定義された全てのテーブルが表示され、手動で選択する必要がある
  - ※ with オプションは 1:n の場合のみ機能する
- table 関数の 3 番目のパラメータ(`extraConfig`?)に対する型は未サポート
  - 実行時には機能するが、型レベルでは機能しない

## Cloudflare D1 でも使用できるのか(確認中)

ローカル D1 のインスタンスをどうやって渡せばいいのかわからず、色々やってみてはいるもののまだ動かせていませんでしたが、

https://x.com/kmkkiii/status/1862913232303710556

自分と同じように D1 で seed が使用できないという Post を発見し、Drizzle ORM の公式アカウントから`bettersqlite3`を使えばできるよ！というリプライがついていました。
あとで試してみてできたら追記します！

https://x.com/DrizzleORM/status/1862946943887937944

## 最後に

ここまで読んでいただいてありがとうございました！
