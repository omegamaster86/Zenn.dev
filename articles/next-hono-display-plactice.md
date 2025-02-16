---
title: "Next.jsとHonoでデータ表示をしてみた"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react","Next.js","Hono"]
published: false
---

# 初めに
何故か会社で鼻水が止まらないオメガマスターです…
そろそろ花粉症の季節なので、リモート希望です（叶わぬ夢）
さて今回は巷で話題の[Hono](https://hono.dev/docs/getting-started/basic)を使ってみました。
使ってみたと言っても、Honoのデータを表示させるだけなので、初学者向けにはなりますが、
とっかかりとして見ていって頂けたら幸いです。

# フォルダ構成
今回のフォルダ構成は下記の感じです。
珍しいことはなく、apps配下にbackendとfrontendを分けています。
backendにはHonoをインストール、
frontendにはNext.jsをインストールしています。
インストールした時から新しく追加したファイルは殆どのないので、HonoとNext.jsをインストールしたら、ほぼ同じような感じになります。
```
.
└── apps
    ├── backend
    │   ├── .next
    │   ├── app
    │   │   ├── api
    │   │   │   └── [...route]
    │   │   │       └── route.ts
    │   │   ├── layout.tsx
    │   │   ├── page.tsx
    │   └── tsconfig.json
    │   └── vitest.config.ts
    │   ├── package.json
    │   ├── node_modules
    │   ├── .gitignore
    │   ├── next-env.d.ts
    │   ├── README.md
    │   ├── tsconfig.json
    │   ├── yarn.lock
    │   └── next.config.js
    └── frontend
        ├── .next
        ├── app
        │   ├── layout.tsx
        │   ├── page.tsx
        │   ├── server.tsx
        │   └── global.css
        ├── node_modules
        ├── .gitignore
        ├── next-env.d.ts
        ├── README.md
        ├── tsconfig.json
        ├── yarn.lock
        ├── next.config.js
        ├── package.json
        ├── postcss.config.mjs
        └── tailwind.config.js
```

# Honoのインストール
ではHonoをインストールしてみましょう。
```
yarn create hono アプリ名
```
その後下記の画像のようにどのテンプレートを使用するか選択します。
![](/images/next-hono-display-plactice/image1.png)
私は今回`Next.js`を選択しました。
その後、記載したアプリ名に遷移し、`yarn`を実行します。
```
cd アプリ名
yarn
```
`yarn`を実行後、下記のコードを実行します。
```
yarn dev
```
