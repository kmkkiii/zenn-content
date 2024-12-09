---
title: 'TILサイトのAstroをv5へアップロードした際の変更点' # 記事のタイトル
emoji: '🚀' # アイキャッチとして使われる絵文字（1文字だけ）
type: 'tech' # tech: 技術記事 / idea: アイデア記事
topics: ['Astro'] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
publication_name: 'mofmof_inc'
---

:::message
この記事は「[mofmof Advent Calendar 2024](https://qiita.com/advent-calendar/2024/mofmof)」10 日目の記事です。
:::

## はじめに

https://astro.build/blog/astro-5/

Astro v5 がリリースされたので、TIL サイトで使用していた Astro v4 からアップグレードしました。
この記事では、アップグレードに伴い変更が必要だったところや v5 で利用できるようになった実験的な機能についてご紹介したいと思います。

https://kmkkiii.github.io/til

## Astro v5 へのアップグレード

https://docs.astro.build/ja/guides/upgrade-to/v5/

Astro v4 から v5 へのアップグレードガイドをみながら進めました。

まずは推奨されているアップグレードコマンドを叩きます。

```bash
> npx @astrojs/upgrade
Need to install the following packages:
@astrojs/upgrade@0.4.0
Ok to proceed? (y)


 astro   Integration upgrade in progress.

      ●  @astrojs/check will be updated to v0.9.4
      ●  @astrojs/rss will be updated to v4.0.9
      ●  @astrojs/tailwind will be updated to v5.1.3
      ▲  astro will be updated to  v5.0.2
      ▲  @astrojs/react will be updated to  v4.0.0

  wait   Some packages have breaking changes. Continue?
         Yes

 check   Be sure to follow the CHANGELOGs.
         astro Upgrade to Astro v5
         @astrojs/react CHANGELOG

 ██████  Installing dependencies with npm...

╭─────╮  Houston:
│ ◠ ◡ ◠  Good luck out there.
╰─────╯
```

アップグレード後に TIL サイトを触ってみて、壊れていたのはページネーションくらいでした。

## paginate()で返される URL が変わった

これまで`/til${page.url.next}`のようにベースパス(`/til`)を自前で書かないといけませんでしたが、astro.config.mjs で`base`が設定してあればベースパスは不要になりました。

https://docs.astro.build/ja/guides/upgrade-to/v5/#changed-urls-returned-by-paginate

## Content Collection から Content Layer へ

Content Collection はレガシーな機能となり、新しく Content Layer が使用できるようになりました。
使用するためにはいくつかやることがあります。

### 設定ファイルの移動

`src/content/content.config.ts`から、`src/content.config.ts`に移動させます。

### コレクション定義の編集

コレクションを選択する`type`プロパティがレガシーとなり、`loader`が使用されるようになりました。
`loader`を定義していない コレクション定義では、内部的に`astro/loaders` の `glob` が使用されます。

https://github.com/kmkkiii/til/blob/83a31ebb93332c8471cfe647399a444e73a38ace/src/content.config.ts#L1-L14

### コレクションの参照に使用していた slug を id に変更

Content Layer コレクションの参照には`id`を使うようになったため、`slug`で参照している箇所は変更する必要があります。dynamic routing ファイルの`[slug].astro` も`[id].astro` に変更します。

### しばらくは従来のコレクションを使用したい場合

これらの変更の準備ができていない場合は、`legacy.collections`フラグを有効にすることで、引き続き従来の Content Collection API の使用を続けることもできます。

## TypeScript の設定変更

これまで`src/env.d.ts`が型推論に使用されていましたが、`.astro/types.d.ts`が使用されるようになりました。
それに伴い、ビルドされたファイルのチェック回避等の推奨される設定を`tsconfig.json`に追記します。

https://github.com/kmkkiii/til/blob/83a31ebb93332c8471cfe647399a444e73a38ace/tsconfig.json#L1-L15

https://docs.astro.build/ja/guides/upgrade-to/v5/#changed-typescript-configuration

## experimental な機能

せっかくなので 新しく利用できるようになった experimental な機能 を 1 つ使用してみました。

### SVG 画像をそのままコンポーネントとして扱える

SVG 画像ファイルをインポートするだけでコンポーネントとして扱えるようになりました。
ネイティブな`<svg>`要素でサポートされている`width`, `height`, `fill`などの属性を props として渡せます。

![TILサイトのフッター](/images/astro_upgrade_to_v5/til-footer.png)
これを利用して、フッターのリンクを SVG 画像のアイコンに変更しました。

https://github.com/kmkkiii/til/blob/83a31ebb93332c8471cfe647399a444e73a38ace/src/components/Footer.astro#L1-L42

## おわりに

ここまで読んでいただいてありがとうございました！
