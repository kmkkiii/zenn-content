---
title: "SVG→PNG→WebPに変換して画像を最適化する" # 記事のタイトル
emoji: "📷" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["svg", "png", "webp", "cwebp", "inkscape"] # タグ。["markdown", "rust", "aws"]のように指定する
publication_name: "mofmof_inc"
published: true # 公開設定（falseにすると下書き）
---

Web サイトやアプリの開発において、画像の最適化はパフォーマンス向上のために重要なタスクです。
特に SVG 形式の画像は、ベクター形式のため拡大や縮小をしても画質が劣化しませんが、ファイルサイズが大きくなりやすいという特徴があります。

開発中のアプリで使用されている画像を WebP に変換して最適化を行ったため、やったことを記録として残しておきます。
また、SVG から PNG に変換するにあたって一部の画像でクリッピングパスの問題があったため対応した内容を共有します。

## PNG 画像を WebP 形式に変換する

WebP は Google が開発した画像形式で、PNG や JPEG に比べてファイルサイズを大幅に削減することができます。
以下のスクリプトでは、現在のディレクトリ配下の PNG 画像を WebP 形式に変換します。
:::message
このスクリプトを実行するには、事前に `cwebp` をインストールする必要があります。
:::

```sh
for file in *.png; do
  cwebp -lossless -metadata icc "$file" -o "${file%.png}.webp"
done
```

このスクリプトは、for ループを使用して現在のディレクトリ以下のすべての PNG 画像に対して変換処理を行います。
`cwebp` コマンドは、各 PNG 画像を入力として受け取り、`-lossless` オプションを指定して可逆圧縮を行い、`-metadata icc` オプションで ICC プロファイルを保持するように指示しています。
出力ファイルは、元のファイル名の拡張子を .webp に変更したものになります。

## SVG 画像を PNG 形式に変換する

次に、SVG 画像を PNG 形式に変換するスクリプトです。
:::message
このスクリプトを実行するには、事前に `Inkscape` をインストールする必要があります。
:::

```sh
for file in *.svg; do
  inkscape --export-type="png" --export-dpi 600 $file
done
```

このスクリプトは、for ループを使用して現在のディレクトリ以下のすべての SVG 画像に対して変換処理を行います。`inkscape` コマンドは、`--export-type` オプションで出力形式を PNG に指定し、`--export-dpi` オプションで解像度を 600dpi に設定しています。

### SVG 画像のクリップ設定が反映されない問題

一部の SVG 画像では、クリップ設定(画像の一部を切り抜いて表示する機能)で x 属性と y 属性が指定されていると、PNG 変換時に正しく反映されない問題が発生しました。

x 属性と y 属性の 代わりに、transform 属性の matrix 関数を使用してクリップ設定を指定することで元の SVG 画像と同じ範囲で切り抜かれた PNG 画像に変換することができました。

```diff:svg
 <g clip-path="url(#clip0_5751_200795)">
-    <rect x="-44" y="-8.00977" width="463" height="347.02" fill="url(#pattern0)"/>
+    <rect width="463" height="347.02" fill="url(#pattern0)" transform="matrix(1, 0, 0, 1, -44, -8.00977)"/>
    <rect opacity="0.16" width="375" height="281" fill="black"/>
  </g>
```

## 参考

https://blog.ideamans.com/2020/08/webp-params-2020.html
