---
title: "Next.js から Solid 2 start mode へ — tech-blog 移行記"
emoji: "🔄"
type: tech
topics: [nextjs, solidjs, supabase, vercel, migration]
published: false
---

GenAi TECH BLOG は **Next.js 16 App Router** から **Solid 2.0 RC + Vite start mode** へ移行した。

シングルページ + Supabase RPC + revalidate だけの構成だったため、Next.js のフルスタック機能は過剰と判断し、Solid 2 start mode を直接採用した。

## このブログの構成

移行前後で変わっていない要件は次のとおり。

- **シングルページ**（トップのみ）
- Supabase RPC 3本（`get_tags`, `get_members`, `get_articles`）
- 86400秒 TTL 相当のキャッシュ + `POST /api/revalidate` によるオンデマンド更新
- SEO メタ（title, OGP, Google Search Console 検証）
- UI: 記事一覧、タグフィルタ、ページネーション、メンバーカルーセル

## なぜ Solid 2 start mode か

| 要件 | Next.js | Solid 2 start mode |
|------|---------|---------------------|
| SSR | Server Component + `force-static` | Vite SSR + route `preload` |
| API Route | `app/api/revalidate/route.ts` | `src/routes/api/revalidate.ts` + middleware |
| データ取得 | Server Component で RPC | `query()` + `preload` + `createMemo` |
| キャッシュ | `revalidate: 86400` + `revalidatePath` | 自前 TTL + `revalidate(key)` |
| ビルド | `next build` | `vite build` → `dist/client` + `dist/server` |

**SolidStart は使わない。** Solid チームは「Start mode replaces SolidStart」と説明しており、Solid 2 移行の正本は `@solidjs/vite-plugin` の `start` オプション。

| 名称 | 実体 |
|------|------|
| **SolidStart**（Vinxi / Nitro） | 旧フレームワーク。Solid 2 移行では経由しない |
| **SolidStart v2** | Solid **1.x** 向け。Node **24** 必須。メンテナンスモード |
| **Solid 2 start mode** | `@solidjs/vite-plugin` の `start`。Node **>=22.12.0** |

## スタック

**主要依存:**

- `solid-js@^2.0.0-rc.9`
- `@solidjs/web`（JSX / SSR）
- `@solidjs/vite-plugin`（start + ssr + serverFunctions）
- `@solidjs/router@^2.0.0-next.26`
- `@solidjs/meta@^1.0.0-next.2`
- `filesystem-routing`, `vite@^8.1.5`, `@tailwindcss/vite`

**使わないもの:** `@solidjs/start`, `vinxi`, Nitro

## 設定の要点

移行で触る設定ファイルは、役割ごとに 1:1 で対応しているわけではない。Next.js は `app/` にページ・レイアウト・API をまとめる一方、Solid 2 start mode は **Vite 設定 + エントリ 3 枚 + ファイルベースルート** に分かれる。

### ファイル対応一覧

| 役割 | Next.js | Solid 2 start mode |
|------|---------|---------------------|
| ビルド・SSR 設定 | `next.config.ts` | `vite.config.ts` |
| ルート定義 | `app/` ディレクトリ（規約ベース） | `src/routes/` + `filesystem-routing` |
| HTML シェル（`<html>` / `<body>`） | `app/layout.tsx` の外側 | `src/Document.tsx` |
| 共通レイアウト・メタ・Router | `app/layout.tsx` の内側 | `src/App.tsx` |
| トップページ | `app/page.tsx`（Server Component） | `src/routes/index.tsx` |
| API Route | `app/api/revalidate/route.ts` | `src/routes/api/revalidate.ts` |
| エッジ / サーバー middleware | `middleware.ts`（任意） | `src/middleware.ts`（API ハンドラ登録） |
| サーバー関数の初期化 | 不要（RSC がそのままサーバー実行） | `src/server-config.ts` |
| Router インスタンス | フレームワーク内蔵 | `src/router.ts` |

---

### `vite.config.ts` ← `next.config.ts`

**Next.js では:** `next.config.ts` に `images`, `rewrites`, `experimental` などを書く。ビルドは `next build` が内部で webpack / Turbopack を呼ぶ。SSR・静的生成・API Route はすべて Next のランタイムが担う。

**Solid 2 では:** Vite の設定ファイルがビルドの中心。`@solidjs/vite-plugin` の `start` オプションで SSR サーバーと Node 本番起動を有効化し、`filesystem-routing` で `src/routes/` をルートに変換する。

```ts
// vite.config.ts
export default defineConfig({
  plugins: [
    tailwindcss(),                    // Next の PostCSS 設定に相当（Tailwind v4 は @tailwindcss/vite）
    solid({
      start: {
        middleware: "./src/middleware.ts",  // API ハンドラの登録先
        node: true,                           // dist/server/node.js を生成（本番は npm start）
      },
      ssr: true,                              // SSR を有効化（Next のデフォルト SSR に相当）
      serverFunctions: {
        configure: "./src/server-config.ts",  // "use server" 関数のサーバー側初期化
      },
    }),
    fileRoutes({ httpMethods: true }),  // routes/api/*.ts を HTTP メソッド別ハンドラに変換
  ],
});
```

| オプション | Next.js での相当 | このプロジェクトでの用途 |
|-----------|-----------------|------------------------|
| `start.node` | `next start` の Node サーバ | `node dist/server/node.js` で本番起動 |
| `start.middleware` | `middleware.ts` + Route Handlers | `POST /api/revalidate` を Node サーバに載せる |
| `ssr: true` | App Router の SSR / SSG | トップページをサーバー描画 |
| `serverFunctions` | Server Component / Server Actions | `getHomeData()` の `"use server"` を有効化 |
| `fileRoutes` | `app/**/page.tsx`, `route.ts` | `routes/index.tsx` → `/`、`routes/api/revalidate.ts` → `/api/revalidate` |

---

### `src/routes/` ← `app/`

**Next.js では:** `app/page.tsx` が `/`、`app/api/revalidate/route.ts` が `/api/revalidate` になる。ディレクトリ名とファイル名が URL に直結する（App Router の規約ルーティング）。

**Solid 2 では:** `filesystem-routing` が `src/routes/` を走査し、`virtual:file-routes` として Vite に注入する。ページはデフォルト export、API は `GET` / `POST` などの名前付き export。

```
Next.js                          Solid 2
app/page.tsx          →  /        src/routes/index.tsx
app/api/revalidate/
  route.ts            →  /api/revalidate   src/routes/api/revalidate.ts（export const POST）
```

ルートごとのデータ取得は `export const route = { preload: ... }` で宣言する。Next の `page.tsx` が async Server Component だった部分に相当する（詳細は後述のデータ取得セクション）。

---

### `src/Document.tsx` ← `app/layout.tsx`（HTML 外側）

**Next.js では:** `app/layout.tsx` が `<html>` / `<head>` / `<body>` を含む。`metadata` export や `next/font` もここに置くことが多い。

**Solid 2 では:** HTML ドキュメントの骨格だけを `Document.tsx` に分離する。SSR 時にサーバーがこのコンポーネントで `<html>` を描画し、クライアントでは `<HydrationScript />` でハイドレーション用スクリプトを差し込む。

```tsx
// Document.tsx — Next の layout.tsx から <html>〜<body> 部分を切り出したイメージ
export default function Document(props: ParentProps) {
  return (
    <html lang="ja">
      <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <link rel="icon" href="/favicon.ico" />
        <HydrationScript />
      </head>
      <body>{props.children}</body>
    </html>
  );
}
```

SEO メタ（title, OGP）は Document ではなく `App.tsx` 側の `@solidjs/meta` に置く。Next の `export const metadata = { ... }` に相当する。

---

### `src/App.tsx` ← `app/layout.tsx`（共通 UI 内側）

**Next.js では:** `layout.tsx` の `{children}` 周りにヘッダー・フッター・フォント・グローバル CSS を置く。全ページ共通のラッパー。

**Solid 2 では:** `App.tsx` がアプリのルートコンポーネント。`<Router>` をマウントし、サイト共通のメタタグ・グローバルスタイル・`<Loading>` フォールバックをここで定義する。各ルートの `children` は Router 経由で流れ込む。

```tsx
// App.tsx — layout.tsx の {children} より外側（Router + 共通メタ）
export default function App() {
  return (
    <Router>
      {(props) => (
        <>
          <Title>GenAi TECH BLOG | ...</Title>
          <Meta name="description" content="..." />
          {/* OGP, Twitter Card, google-site-verification など */}
          <div class="font-sans antialiased min-h-screen ...">
            <Loading>{props.children}</Loading>
          </div>
        </>
      )}
    </Router>
  );
}
```

| Next.js | Solid 2 |
|---------|---------|
| `export const metadata = { title, openGraph, ... }` | `<Title>` + `<Meta>`（`@solidjs/meta`） |
| `import "./globals.css"` | `import "./app.css"` |
| `{children}` をそのまま描画 | `<Router>` → `props.children` |

---

### `src/routes/index.tsx` ← `app/page.tsx`

**Next.js では:** `app/page.tsx` を async Server Component にし、関数本体で `await supabase.rpc(...)` して JSX を返す。`export const revalidate = 86400` で ISR を宣言できる。

**Solid 2 では:** 通常の関数コンポーネント。データは `query()` で包んだサーバー関数（`getHomeData`）を `route.preload` で先行取得し、`createMemo(() => getHomeData())` で参照する。未解決中は `<Loading>` がフォールバック UI になる。

```tsx
// routes/index.tsx
export const route = {
  preload: () => { void getHomeData(); },  // Next の async page 冒頭の await に相当
};

export default function Home() {
  const data = createMemo(() => getHomeData());
  return (
    <main>
      <Header />
      <Loading>
        <Show when={data()}>{(home) => /* Members, Articles */}</Show>
      </Loading>
      <Footer />
    </main>
  );
}
```

ページ固有の UI（Header, Hero, Articles など）は Next 時代と同様コンポーネントに分割する。配置先が `app/components/` から `src/components/` に変わるだけ。

---

### `src/routes/api/revalidate.ts` ← `app/api/revalidate/route.ts`

**Next.js では:** `route.ts` に `export async function POST(request)` を書く。`revalidatePath('/')` で ISR キャッシュを破棄する。

**Solid 2 では:** 同名パスに `export const POST: APIHandler` を書く。自前 TTL キャッシュの `invalidateHomeDataCache()` と、Router の `revalidate(getHomeData.key)` を呼ぶ。Next の `revalidatePath` に直接相当する API はない。

```ts
// routes/api/revalidate.ts
export const POST: APIHandler = async ({ request }) => {
  // REVALIDATE_TOKEN 検証
  invalidateHomeDataCache();
  revalidate(getHomeData.key);
  return json({ revalidated: true, path: "/", now: Date.now() }, 200);
};
```

---

### `src/middleware.ts`

**Next.js では:** プロジェクトルートの `middleware.ts` でリクエストを横断的に処理する（認証リダイレクト、ヘッダー付与など）。tech-blog では移行前も revalidate 専用の middleware は不要だった。

**Solid 2 では:** start mode の `middleware` オプションで指定するファイル。ここでは `filesystem-routing/api` の `createAPIHandler` を登録し、`/api/*` へのリクエストを Node サーバで受ける。

```ts
// middleware.ts
import routes from "virtual:file-routes";
import { createAPIHandler } from "filesystem-routing/api";

export default [createAPIHandler(routes)];
```

Next の Edge Middleware とは別物。ページ SSR 用の前処理ではなく、**API Route を start mode サーバに載せるためのフック**として使っている。

---

### `src/server-config.ts`

**Next.js では:** 相当ファイルなし。Server Component はビルド時にサーバー境界が自動で切られる。

**Solid 2 では:** `"use server"` 付き関数（サーバー関数）のサーバー側初期化を行う。`@solidjs/router` の Flight データ収集を Router に紐づける。

```ts
// server-config.ts
import { createFlightDataCollector } from "@solidjs/router/server";
import { configureServerFunctionsServer } from "@solidjs/web/server-functions/server";
import { Router } from "./router";

configureServerFunctionsServer({
  collectFlightData: createFlightDataCollector(Router),
});
```

`getHomeData()` が `"use server"` でサーバー実行され、クライアントの `createMemo` から透過的に呼べるのは、この設定と `vite.config.ts` の `serverFunctions` がセットになっているため。

---

### `src/router.ts`

**Next.js では:** ルーティングはフレームワーク内蔵。`app/` のファイル構造がそのままルートテーブルになる。

**Solid 2 では:** `@solidjs/router` のインスタンスを明示的に生成する。`virtual:file-routes` から `fileRoutes()` でルート定義を組み立て、`App.tsx` の `<Router>` に渡す。

```ts
// router.ts
import { pageRoutes } from "virtual:file-routes";
import { createRouter } from "@solidjs/router";
import { fileRoutes } from "@solidjs/router/fs";

export const Router = createRouter({ routes: fileRoutes(pageRoutes) });
```

Next における「規約ルーティングの結果をコードで触る」場所が、この 1 ファイルに集約される。

## React / Next.js → Solid 2 の変換

| React / Next.js | Solid 2 |
|-----------------|---------|
| `useState` | `createSignal` |
| `useEffect` | `onSettled` + return cleanup |
| `className` | `class` |
| `next/image` | `<img>` |
| `metadata` export | `@solidjs/meta` |
| `NEXT_PUBLIC_*` | `VITE_*` |
| Server Component の async | `query()` + `"use server"` |
| `Suspense` | `Loading` |
| `JSX` 型 from `react` | from `@solidjs/web` |

### データ取得

Server Component の async 関数から、Router 2 の `query()` + `preload` + `createMemo` へ置き換えた。

```tsx
// Before: Next.js Server Component
export default async function Home() {
  const { data: tags } = await supabase.rpc("get_tags");
  // ...
}

// After: Solid 2 start mode
export const route = {
  preload: () => { void getHomeData(); },
};

export default function Home() {
  const data = createMemo(() => getHomeData());
  return (
    <Loading>
      <Show when={data()}>{(home) => /* ... */}</Show>
    </Loading>
  );
}
```

Router 2 では `preload` は **開始だけ**（戻り値を props で読まない）。未解決 UI は `<Loading>` で待つ。

### キャッシュ

Next.js の ISR に相当する処理を自前実装した。start mode に ISR がないため、サーバー関数側のメモリキャッシュとして残している。

```ts
// src/lib/home-cache.ts — TTL 86400s
export function readHomeCache<T>(): T | null { /* ... */ }
export function writeHomeCache<T>(value: T) { /* ... */ }
export function invalidateHomeDataCache() { entry = null; }
```

再検証は `POST /api/revalidate` で `invalidateHomeDataCache()` + `revalidate(getHomeData.key)` を呼ぶ。

## 現行アーキテクチャ

```mermaid
flowchart LR
  route["routes/index.tsx\npreload + createMemo + Loading"] --> query["getHomeData()\nquery + TTL cache"]
  query --> supabase["Supabase RPC x3"]
  api["POST /api/revalidate"] --> invalidate["invalidateHomeDataCache\n+ revalidate(key)"]
  invalidate --> query
  route --> components["components/*.tsx"]
```

**ビルド・起動:**

```bash
npm run dev     # vite
npm run build   # dist/client + dist/server/
npm start       # node dist/server/node.js
```

**環境変数:**

| 変数 | 用途 |
|------|------|
| `VITE_SUPABASE_URL` | クライアント公開 |
| `VITE_SUPABASE_ANON_KEY` | クライアント公開 |
| `REVALIDATE_TOKEN` | サーバー秘密（`VITE_` を付けない） |

## デプロイ上の注意

### Vercel

Framework Preset は **Other**、Node.js **22.12+**。

Next.js Preset のままだと `No Next.js version detected` で Preview が失敗する。

公式の Solid 2 start-mode Vercel アダプタは移行時点で未確認。**Vercel への自動デプロイは blocker になり得る**。Nitro preset は使えず、start mode の Node サーバ契約（`handleRequest`）を自分で載せる必要がある。

### OGP 画像

`public/ogp.png` がリポジトリに存在しないため、image メタは省略している。404 を避けるための判断。

## 移行を振り返って

### うまくいったこと

- **要件の絞り込み**: シングルページ + RPC + revalidate だけなら、Next.js の機能を全部使う必要がなかった
- **start mode 直採用**: SolidStart / Vinxi を経由せず、Vite 一本で SSR + サーバー関数 + API を構成できた
- **`solid-migration-assistant`**: 移行前に変更点を洗い出し、移行後に検出ゼロを確認できた
- **自前 TTL**: Next.js ISR と同等のキャッシュ戦略を start mode でも維持できた

### ハマりどころ

- **SolidStart と start mode の名前**: 別物。Solid 2 移行では start mode が正本
- **Tailwind + Vite 8 SSR**: PostCSS 単体が壊れたため `@tailwindcss/vite` を追加
- **Vercel**: start mode の Node サーバ契約を自分で載せる必要がある

## まとめ

tech-blog は **Next.js 16 App Router** から **Solid 2.0 RC + Vite start mode** へ直接移行した。

同規模のシングルページ + Supabase 構成なら、Next.js を維持するより Solid 2 start mode の方がシンプルになる、というのが今回の結論。

## 参考リンク

- [org-genai/tech-blog](https://github.com/org-genai/tech-blog)
- [Solid 2.0 Migration Guide](https://github.com/solidjs/solid/blob/next/documentation/solid-2.0/MIGRATION.md)
- [Solid 2.0 RC: The Big Reveal](https://www.solidjs.com/blog/solid-2-0-rc-the-big-reveal)
