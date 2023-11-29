---
pubDatetime: 2023-11-28T22:35:00+09:00
title: Astroを試してみる
postSlug: astro-getting-started
featured: false
draft: false
tags:
  - blog
  - astro
description:
  まずはAstroを触ってみる話
---

Astoroでのブログを作成するために、まずAstroの動作確認をしてみる。

## Astroのプロジェクトの雛形を作成

- [Astro公式ページのGetting Started](https://docs.astro.build/en/getting-started/) に書かれているように、 CLIでプロジェクトの雛形を作成する

```sh
yarn create astro
```

- CLIのウィザードに従って選択していけば雛形が作成できる
- 特徴的な質問と回答は以下

- How would you like to start your new project?
  - Use blog template を選択

- Do you plan to write TypeScript?
  - Yes を選択

- How strict should TypeScript be?
  - Strict (recommended) を選択

- プロジェクトの雛形が作られる

## 起動確認

- 以下のコマンドで開発モードで起動できる

```sh
yarn dev
```

- ブラウザで、 `http://localhost:4321/` にアクセスするとBlogの雛形が動いている

![Blogの雛形](/assets/astro-getting-started/blog-template.png)

## ブログの記事の編集を確認

- プロジェクトのディレクトリの `src/content/blog` にブログの記事になるコンテンツが置かれている
- 以下はサンプルで登録されているコンテンツ
- これをコピーして新しい記事を作成していく
- 記事はMarkdownで書いていく

```md
---
title: 'First post'
description: 'Lorem ipsum dolor sit amet'
pubDate: 'Jul 08 2022'
heroImage: '/blog-placeholder-3.jpg'
---

Lorem ipsum dolor sit amet, consectetur adipiscing elit,
sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
```

- `---` に囲まれた部分がページのメタ情報を書くところ
- pubDateに記事の投稿日時をセットしておくと、その値を使って記事の順序がソートされる
- その下に普通にMarkdownを書けば記事は完成

## Blogのカスタマイズ

- レイアウトなどを調整したい場合は `src/layouts` の中のファイルをいじれば良い
- ブログの記事以外のページをいじりたい場合は、 `src/pages` の中のファイルを編集する
  - 例: トップページを編集する場合 `src/pages/index.astro` を編集すれば良い
- Astroページは `.astro` の拡張子のファイルを使って書く
  - `.html` も置けるが、コンポーネントを使いたい場合が多いので `.astro` を使っておく方が無難
  - `.astro` ファイルは Astroのコンポーネントと同じ
  - コンポーネントでは、HTMLを書いたり、他のコンポーネントをインポートしたりデータを読み込んだり、、、さまざまなことができる

## 感想

- 特に悩むことなくブログの雛形が作れた
- CLIでセットアップされるBlogテンプレートでも、ちゃんとレスポンシブになっていたり、Markdownが使えたりするのでそのままでも良いかもしれない
