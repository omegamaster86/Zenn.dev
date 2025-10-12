---
title: "Next.jsとHonoでデータ表示をしてみた"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react","Next.js","Hono","TypeScript","初学者"]
published: true
published_at: 2025-02-17 10:00
publication_name: "genai"
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
その後、localhostがたちあがり、`Hello from Hono!`と表示されます。

# Honoに記載のあったコード
```:apps/backend/api/[...route]/route.ts
import { Hono } from 'hono'
import { handle } from 'hono/vercel'

export const runtime = 'edge'

const app = new Hono().basePath('/api')

app.get('/hello', (c) => {
  return c.json({
    message: 'Hello from Hono!'
  })
})

export const GET = handle(app)

```
`export const runtime = 'edge'`の意味はVercel やその他の対応プラットフォームでは、runtime の値を 'edge' にすることで、サーバーレスエッジ環境上でコードを実行するように指示できます。
効果としてエッジ環境は、リクエスト元に近い場所でコードを実行するため、低レイテンシで高速なレスポンスが期待できます。
```apps/backend/api/[...route]/route.ts
app.get('/hello', (c) => {
  return c.json({
    message: 'Hello from Hono!'
  })
})
```
こちらに関してa`pp.get`は、Honoのルーティングメソッドであり、第一引数にエンドポイントのパス（この場合は '/hello'）を指定します。
`c`はHonoのコンテキストオブジェクトで、リクエスト情報やレスポンスを扱うためのメソッドが含まれています。
`const app = new Hono().basePath('/api')`としているため、実際のエンドポイントは`/api/hello`になります。

ちなみに
```apps/backend/api/[...route]/route.ts
let blog = [
  {
    id: 1,
    title: 'Hello World',
    body: 'Hello World',
  },
  {
    id: 2,
    title: 'Hello World2',
    body: 'Hello World2',
  },
  {
    id: 3,
    title: 'Hello World3',
    body: 'Hello World3',
  },
]

app.get('/blog', (c) => c.json({ post: blog }))
```
を追加すると、localhost:3000/api/blogにアクセスすると、json形式で記載したblogの内容が表示されます。

idに応じて動的に内容を取得したい場合、下記のように記載します。
```apps/backend/api/[...route]/route.ts
app.get('/blog/:id', (c) => {
  const id = c.req.param('id')
  const post = blog.find((post) => post.id === id)
  
  if(post) {
    return c.json( post );
  } else {
    return c.json({ message: 'Not Found' }, 404);
  }
})
```

# フロントエンド側
```:server.ts
'use server'
export const fetchData = async () => {
  const res = await fetch('http://localhost:3000/api/hello')
  const data = await res.json()
  return data
}
```
```:page.tsx
import { fetchData } from './server'
export default async function Home() {
    const data = await fetchData()

  return (
    <div className="flex flex-col items-center justify-center min-h-screen py-2">
      <p>{data.message}</p>
    </div>
  );
}
```
これでバックエンドのroute.tsでバックエンドに保存してある`Hello from Hono!`をフロントエンドで表示することができました。

# 注意点
corsの設定をしないと、フロントエンドからバックエンドにアクセスできないので、以下のように設定する必要があります。
:::message alert
ガバガバのcors設定なので、真似しないでください
:::
```apps/backend/api/[...route]/route.ts
import { Hono } from 'hono'
import { handle } from 'hono/vercel'
import { cors } from 'hono/cors'

export const runtime = 'edge'

const app = new Hono().basePath('/api')

app.use('/*' , 
  cors({
    origin: ['http://localhost:3001'],
    allowMethods: ['GET', 'HEAD', 'POST', 'PUT', 'DELETE', 'CONNECT', 'OPTIONS', 'TRACE', 'PATCH'],
    allowHeaders: ['Content-Type', 'Authorization'],
    credentials: true,
  })
)

app.get('/hello', (c) => {
  return c.json({
    message: 'Hello from Hono!'
  })
})

export const GET = handle(app)
```

#最後に
今回は型を省略しましたが、次回では型つきで実装します。
そしてこの記事がHonoを使用する誰かの役に立てば幸いです。
またこの記事をスタートとして、もっと他の方の記事を見て、より良い開発体験が出来ると幸いです。