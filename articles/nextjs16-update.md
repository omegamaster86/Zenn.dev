---
title: "Next.js16公式とβ版について調査したんじゃ"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Next.js"]
published: true
published_at: 2025-10-27 09:00
publication_name: "genai"
---
# 初めに
10月に入り寒い日も続きますね〜同時に様々な技術のカンファレンスもあり、情報を追うが大変ですが頑張って追いかけましょう〜（個人的にはもっと分散してもらえると助かりますが…）
今回のリリースでは、ビルドツールのTurbopack、キャッシュ機能、フレームワークのアーキテクチャに対して大規模な改良が加えられています。恐らくみなさんが気になるのは、Cache Components、Next.js Devtools MCPの２つですかね〜
この2つを重点的に見つつ、他の情報も見ていきましょう！

# Next.js Conf 25の動画
https://youtu.be/IdIgkiDu-94

# Cache Components
Cache ComponentsはNext.js 16で追加された新しいキャッシュの仕組みで、ページやコンポーネント、関数レベルでの柔軟なキャッシュ制御を可能にします。従来のApp Router（appディレクトリ）の自動キャッシュとは異なり、Cache Componentsは`use cache`ディレクティブを用いたオプトイン方式のキャッシュモデルです。。デフォルトではすべての動的なコード（ページ、レイアウト、APIルート内の処理）はリクエスト時に実行されますが、`use cache`を宣言した部分についてはビルド時にキャッシュキーが自動生成され、実行結果がキャッシュされます。
Cache Componentsの導入により、Partial Pre-Rendering (PPR) のコンセプトが完成されたと公式で説明されています。
https://nextjs.org/blog/next-16#cache-components

PPRとは、これまで各ページを静的または動的のどちらか一方でしか生成できなかった制約を打ち破り、ページ内の一部分だけを動的レンダリングに切り替える手法です。Suspenseを用いて静的ページの一部を動的化することで、完全静的ページの高速初期表示を維持しつつ動的データも扱えるようになりました。Cache Componentsにより、このPPRの考え方が正式に組み込まれ、開発者は必要な部分だけを明示的にキャッシュしつつ、その他はリクエスト毎に最新データを取得するよう柔軟に選択できます。PPRに関してより詳しい詳細を知りたい方はこちらをご覧ください。
https://vercel.com/blog/partial-prerendering-with-next-js-creating-a-new-default-rendering-model

Cache Componentsを有効化するには、`next.config.ts`で以下のように設定します。
```:next.config.ts
const nextConfig = {
  cacheComponents: true,
};
export default nextConfig;
```
:::message
ベータ版で導入されていたexperimental.ppr関連の設定は、このCache Components設定に置き換えられたため削除されました。
またCache Componentsの詳細な使用方法については、今後公式ドキュメントやブログで追加情報が提供される予定と公式が言っているので、もうしばらく待ってみましょう。
:::

# Next.js Devtools MCP
Next.js Devtools MCP（Model Context Protocol）と呼ばれる新機能も導入されました。これはAIを活用した開発支援ツールで、開発中のアプリケーションにコンテキスト情報を提供し、より高度なデバッグや問題解析を支援するものになっています。
Next.js DevTools MCP は AI エージェントに次の機能を提供します。
- Next.js の知識: ルーティング、キャッシュ、レンダリングの動作
- 統合ログ:コンテキストを切り替えることなくブラウザとサーバーのログを取得
- 自動エラーアクセス: 手動でコピーすることなく詳細なスタックトレースを取得
- ページ認識：アクティブルートのコンテキスト理解
これにより、AI エージェントは開発ワークフロー内で直接、問題を診断し、動作を説明し、修正を提案できるようになります。

# Next DevTools MCP（外部パッケージ）とNext.js MCPサーバー（組み込み）の比較
私がNext DevTools MCPとNext.js MCPサーバーの違いがいまいちわからなかったので、少しお付き合いください。
Next.js公式ドキュメントによれば、Next.jsはこのMCPに対応する2種類のサーバーを提供しており、「Next.js MCPサーバー（組み込み）」と「Next DevTools MCP（外部パッケージ）」の両方を併用することで最良のエージェント支援環境が実現できるとされています。
### 目的と役割の違い
Next.js MCPサーバー（組み込み）
Next.js 16以降に標準搭載されたMCPサーバーです。**開発サーバー内部で動作し、アプリケーションの内部状態をリアルタイムにエージェントへ公開する役割を担います。**これにより、エージェントは実行中アプリの状態、ページルートやメタデータ、ビルドエラー・ランタイムエラーとログ、Server Actionsやコンポーネント階層などを直接取得・検査できます。つまり組み込みMCPサーバーは、開発中のアプリケーション内部を詳しく観察しリアルタイムな診断情報を提供することを目的としています。

Next DevTools MCP（外部パッケージ）
next-devtools-mcpという名称で提供される独立したNPMパッケージのMCPサーバーです。**Next.jsアプリに対して高レベルな開発支援ツールやガイダンスを提供する目的があります。**具体的には、Next.js公式ドキュメントの知識ベースへのクエリ機能、Next.js 16への移行・アップグレード支援（コードモッドによる自動コード変換）機能、Cache Componentsの設定方法ガイド、およびブラウザテスト支援（Playwright MCPとの統合によるページ動作検証）などを含みます。またNext DevTools MCPは実行中のNext.js開発サーバーを自動検出し、組み込みMCPサーバーが提供する各種内部情報取得ツール（get_errorsやget_logs等）に直接アクセスする能力も持ちます。つまり、**外部パッケージ版MCPサーバーはNext.js全般に関する知識やベストプラクティスの提供、及び必要に応じて組み込みサーバーからのデータ取得を通じて開発者への指針や支援を与える**ことを目的としています。

### 開発ワークフローにおける位置づけ
Next.js MCPサーバー（組み込み）
Next.jsの開発サーバー(npm run dev)に内蔵されて動作するため、開発者が特別な操作をしなくても開発時に自動的にMCPサーバーが起動します。Next.js v16以上ではこの機能がデフォルトで有効化されており、開発サーバーを立ち上げるだけでMCPサーバーが利用可能です。
Next DevTools MCP（外部パッケージ）
開発サーバーとは別プロセスで動作する外部ツールであり、必要に応じて個別に起動・接続します。こっちがみなさんがよく使用するMCPです。下記の設定が必要なやつ。
```
{
  "mcpServers": {
    "next-devtools": {
      "command": "npx",
      "args": ["-y", "next-devtools-mcp@latest"]
    }
  }
}
```
### 提供される機能やAPIの違い
Next.js MCPサーバー（組み込み）の機能/API
Next.js MCPサーバーはエージェントに対して開発中のアプリ内部を操作・問い合わせるための各種「ツール」APIを提供します。公式ドキュメントによれば、主なツールAPIとして以下が利用可能です。
- get_errors: 現在発生しているビルドエラー・ランタイムエラー・型エラーを取得
- get_logs: 開発サーバーのログやコンソール出力を取得
- get_page_metadata: 指定したページに関するルートや使用コンポーネント、レンダリング情報などのメタデータを取得
- get_project_metadata: プロジェクト全体の構造や設定に関するメタデータを取得
- get_server_action_by_id: 指定IDのServer Actionを検索しデバッグ情報を取得

これらのAPIにより、エージェントはアプリケーションの現在の状態や構成を詳しく把握できます。例えばエラー取得やログ参照、ページ構造の解析といった操作が可能です。
https://nextjs.org/docs/app/guides/mcp#:~:text=,ID%20for%20debugging%20and%20inspection

Next DevTools MCP（外部パッケージ）の機能/ツール
Next DevTools MCPはNext.js開発を支援する高レベルな機能を提供します。主な機能は以下の通りです
- Next.js知識ベースクエリ: Next.js公式ドキュメントやベストプラクティスに基づく情報を検索・参照する機能（エージェントが質問に答える際に公式情報源を利用可能）
- 移行・アップグレード支援ツール: Next.jsのバージョンアップ（特にv16への移行）に際し、コード修正の自動化（コードモッド）や移行手順のガイドを提供
- Cache Components設定ガイド: 新しいキャッシュコンポーネントの導入や設定方法について助言やベストプラクティスを提示
- ブラウザテスト統合: 【Playwright MCP】との連携により、開発中のアプリページを実際にブラウザ上で検証・テストする機能

さらに、Next DevTools MCPは上記の独自機能に加えて組み込みMCPサーバーの低レベルAPIにも橋渡しします。すなわち、外部サーバー経由でget_errors等を呼び出し、取得した内部データをもとにより高度な分析や提案を行うことができます。このように、外部パッケージ版はNext.jsに関する知識提供や自動化支援にフォーカスしつつ、必要に応じて組み込み版の提供するデータにもアクセスする仕組みです。

### ユースケース（適用シーン）の違い
Next.js MCPサーバー（組み込み）に適した用途
**リアルタイムなデバッグや内部状態の診断が必要なケース**で威力を発揮します。例えば「現在アプリにどんなエラーが起きているか？」とエージェントに質問すると、組み込みMCPサーバーのget_errorsツールを通じて最新のビルドエラーやランタイムエラーを収集し、エージェントがそれを分析して修正提案を返すことができます。実際、**MCP対応エージェントはNext.js MCPサーバーから取得したエラー情報をもとに原因を特定し、具体的な解決策を提示してくれます。**また、現在のページ構成やコンポーネントツリーを調べて「どのファイルに新しい機能を追加すべきか」といったプロジェクト文脈に沿ったコード提案を行う際にも、組み込みサーバーから得られる詳細なメタデータが有用です。要するに、現在進行中のアプリの状態把握や不具合調査といったユースケースに適しています。

Next DevTools MCP（外部パッケージ）に適した用途
**Next.jsの知識に基づくガイダンス提供やコードベース全体への支援が必要なケースに向いています。**例えば「このプロジェクトをNext.js 16にアップグレードしたい」と依頼すると、エージェントはNext DevTools MCPを用いて現在のプロジェクトのNext.jsバージョンや構成を分析し、自動コード変換ツール（コードモッド）による移行や、必要な破壊的変更への対処手順を段階的に案内してくれます。また「App Routerでどのような場合に'use client'を使うべきか？」といったフレームワークの概念に関する質問に対しては、Next.jsの知識ベースを参照して公式ドキュメントに裏付けされた解説を行い、場合によってはプロジェクトのコード例も示してくれます。このように、Next DevTools MCPはフレームワークのベストプラクティスや設計指針の提示、開発効率化のための自動化（テスト実行や移行支援など）に適したユースケースで活躍します。

なんとなくイメージができましたが、Next DevTools MCP（外部パッケージ）はcursorの@webでNextの公式ドキュメントを参照させ、検索させるのと同じイメージなので、絶対に必要かと言われると個人的には微妙な気がします。まぁ公式的には両方導入する方が良いと記載があるので、両方入れておけば無難ですね。
どうせNext.js MCPサーバー（組み込み）は自動的に適応されますし。

# ミドルウェアの`proxy.ts`への移行
Next.js 16では、カスタムミドルウェアとして機能していた`middleware.ts`ファイルが非推奨となり、代わりにproxy.tsファイルを使用するよう変更が加えられました。proxy.tsはNode.jsランタイム上で実行されるため、アプリケーションのネットワーク境界を明確化する役割を果たします。（Edge Runtimeではなく常にNode.jsで動作するようです）
:::message
このmiddleware.tsファイルは Edge ランタイムの使用例では引き続き使用できますが、非推奨となっており、将来のバージョンでは削除される予定です。by公式より
:::
使い方としては、プロジェクト内のmiddleware.tsファイルを**proxy.tsにリネーム**し、エクスポートされている関数名もproxyに変更するだけです。（ロジックの内容はそのまま使用可能）
例えば、以下のように既存のミドルウェアをリネームします
```
// 旧: middleware.ts → 新: proxy.ts
export default function proxy(request: NextRequest) {
  return NextResponse.redirect(new URL('/home', request.url));
}
```

https://nextjs.org/docs/app/getting-started/proxy

# ログの改善
ビルドおよび開発サーバー実行時のログ出力が強化され、処理の内訳や所要時間がより明確に表示されるようになりました。開発中にターミナル上に表示されるリクエストログでは、新たに各処理がどのフェーズで時間を消費しているかが示されます。
コンパイル: ルーティングとコンパイル
レンダリング: コードの実行とReactレンダリング
![](/images/nextjs16-update/1.png)
ビルド機能も拡張され、時間がかかった箇所も表示されるようになりました。ビルドプロセスの各ステップと、完了にかかった時間が表示されるようになりました。
![](/images/nextjs16-update/2.png)

# Next.js16のβ版での発表
### Turbopackのデフォルト採用とパフォーマンス向上
Turbopackは開発ビルドと本番ビルドの両方で安定稼働を達成し、すべての新規Next.jsプロジェクトのデフォルトバンドラーとなりました。ベータリリース以降、採用は急速に拡大しており、Next.js 15.3以降では開発セッションの50%以上と本番ビルドの20%が既にTurbopack上で実行されています。
Turbopack を使用すると、次のことが期待できます。
- 2～5倍高速なプロダクションビルド
- 最大10倍高速な高速リフレッシュ
これらのパフォーマンス向上をすべてのNext.js開発者に提供するため、Turbopackがデフォルトになりました。特段設定は不要です。カスタムWebpack設定を使用しているアプリの場合は、以下のコマンドを実行することで引き続きWebpackを使用できます。
```
next dev --webpack
next build --webpack
```
### Turbopack ファイル システム キャッシュ
Turbopack は開発時にファイルシステム キャッシュをサポートするようになりました。これにより、実行間でコンパイラ成果物がディスクに保存され、特に大規模プロジェクトでは再起動後のコンパイル時間が大幅に短縮されます。
```: nextConfig
const nextConfig = {
  experimental: {
    turbopackFileSystemCacheForDev: true,
  },
};
 
export default nextConfig;
```
すべての社内 Vercel アプリではすでにこの機能が使用されており、大規模なリポジトリ全体で開発者の生産性が著しく向上していることが確認されているようです。by公式

### create-next-appの簡素化
- App Routerの採用: 新しいルーティングシステム（appディレクトリ構造）を既定で利用
- TypeScriptサポート: プロジェクトがTypeScriptファーストで構成され、型定義が最初から利用可能
- Tailwind CSSの統合: 人気のCSSフレームワークであるTailwind CSSが標準でセットアップ
- ESLintの組み込み: コード品質ツールとしてESLintが設定済みでプロジェクトが開始
これらにより、開発者はプロジェクト生成直後から最新のベストプラクティスが反映された環境で開発を開始できます。
![](/images/nextjs16-update/3.png)

### npx create-next-appを実行した場合
![](/images/nextjs16-update/4.png)
Yes, use recommended defaultsを選択すると下記の設定で
TypeScript, ESLint, Tailwind CSS, App Router, Turbopack
No, reuse previous settingsを選択すると下記の設定で
TypeScript, Biome, Tailwind CSS, App Router, Turbopack
PJを開始できます。
Biomeを使用したい方はnpx create-next-appの方が良いですね

### React Compilerサポートの安定化
Reactコンパイラ1.0のリリースに伴い、Next.js 16ではReactコンパイラの組み込みサポートが安定しました。Reactコンパイラはコンポーネントを自動的にメモ化し、手動によるコード変更を一切行わずに不要な再レンダリングを削減します。
Next.js 15.xではexperimental.reactCompilerオプションとして試験提供されていましたが、16では設定項目reactCompilerが正式に安定版へ昇格しました。`next.config.ts`で以下のように指定することでReact Compilerを有効にできます。
```nextConfig
const nextConfig = {
  reactCompiler: true,
};
export default nextConfig;
```
React Compilerを有効にすると、ビルド時および開発中のコンパイル時間が増加する可能性がある点には注意が必要です。これは、React CompilerがBabelを介してコードを解析・変換するためで、特に大規模アプリケーションではビルド所要時間が長くなることがあります。現時点ではデフォルト無効とされていますので、使用する場合には`npm install babel-plugin-react-compiler@latest`を実行してください。

### レイアウトの重複読み込みの排除（Layout Deduplication）
複数のページ/リンク間で共有されるレイアウトコンポーネントは、一度だけダウンロードすれば済むようになりました。例えば、50個の商品リンクがあるページでそれらすべてが同じレイアウトを共有する場合、従来は各リンクの事前フェッチで同じレイアウトを重複してダウンロードしていましたが、Next.js 16では共通レイアウトを一度だけ取得し50回分の重複転送を削減します。これによりネットワーク帯域の無駄が大幅に削減されます。

### インクリメンタルなプリフェッチ（Incremental Prefetching）
Next.js 16では事前読み込みの挙動がよりスマートになり、キャッシュにない部分だけをフェッチするように変更されました。具体的には、キャッシュ済みでないデータやコードのみを対象にプリフェッチを行い、既に取得済みの部分は再取得しません。プリフェッチキャッシュは下記のようになります。
- リンクがビューポートから外れるとリクエストをキャンセルします
- ホバー時またはビューポートに再度入ったときにリンクのプリフェッチを優先します
- データが無効になったときにリンクを再プリフェッチします
- キャッシュコンポーネントなどの今後の機能とシームレスに連携します
これらの改良により、ページ遷移時の体感速度が向上し、不要なデータ通信を抑えつつ必要なリソースを先読みすることで全体的な効率が上がります。トレードオフとして、従来よりもプリフェッチのリクエスト数自体は増加する可能性がありますが、総通信量は削減されるため多くの場合に有益の可能性があります。
プリフェッチをオンにしていると、不要なプリフェッチが「エッジリクエスト」を増やす原因となり、料金に影響を与える場合があるので、特段不要なら使用しない方が良いと思います。

### キャッシュAPIの改善(revalidateTag, updateTag, refresh)
#### revalidateTag() の仕様変更
キャッシュに付与したタグに対して再検証（stale-while-revalidate, SWR的な再取得）を行うrevalidateTag関数は、第2引数に「キャッシュ期間プロファイル (cacheLife)」を指定するようになりました。例として、ブログ記事一覧に付けたタグ'blog-posts'を再検証したい場合、次のように使用します.
```
import { revalidateTag } from 'next/cache';

// `'max'`プロファイルを指定（多くの場合はこれを推奨）
revalidateTag('blog-posts', 'max');
```
ここで指定するプロファイルは'max'（最長寿命、長期間有効だがバックグラウンド再検証あり）のほか、'hours'や'days'といった組み込みプロファイル、あるいは{ revalidate: 3600 }のように秒数を直接指定するカスタム設定も可能です。
```
revalidateTag('news-feed', 'hours');
revalidateTag('analytics', 'days');
 
revalidateTag('products', { revalidate: 3600 });
```
'max'プロファイルを使うと、キャッシュされたデータを即座に返しつつバックグラウンドで再検証を行うSWR動作となり、ユーザーには古いデータでもすぐ応答しつつ裏で更新が走ります。
revalidateTag('blog-posts')のように引数1つだけで呼び出していた場合は非推奨となり、自身が発行した書き込みを即座に反映させたい（直後の読み出しで最新化したい）場合には、updateTagの使用が推奨されています。

#### そもそもrevalidateTagって？
「タグ付きでキャッシュされたデータ」を任意のタイミングで再検証（SWR）させるAPIです。`fetch(..., { next: { tags } })` や `unstable_cache(..., { tags })` で付けたタグに対して呼び出します。
いつ使う？
- **DB更新やWebhook受信の直後**に一覧・詳細などの「データキャッシュ」を新鮮化したいとき
- **ISRの待ち時間をスキップ**して、必要な瞬間だけ最新化をトリガーしたいとき
- 同じデータを使う複数ルートを、**URLに依らず横断的に更新**したいとき

どこで呼べる？（サーバーのみ）
Server Actions / Route Handlers / サーバーコンポーネント / Edge Runtime など

何に効く？
`fetch(GET)` の Data Cache と `unstable_cache` の結果。HTMLそのものではなく「データ」のキャッシュが対象

使い方（基本）
1) 取得側でタグを付ける
```ts
// fetch にタグを付ける
const res = await fetch('https://api.example.com/posts', {
  next: { tags: ['posts'] },
});
const posts = await res.json();
```

```ts
// unstable_cache にタグを付ける
import { unstable_cache } from 'next/cache';

export const getPost = unstable_cache(
  async (id: string) => {
    // DBや外部APIから取得
  },
  ['getPost', /* key */],
  { tags: ['posts', (id) => `post:${id}`] }
);
```

2) 変更側で再検証をトリガーする
```ts
'use server';
import { revalidateTag } from 'next/cache';

export async function createPost(input: { title: string }) {
  // 何らかの書き込み
  // await db.post.create({ data: input });

  // SWR再検証（既存キャッシュは即返しつつ裏で更新）
  revalidateTag('posts', 'max');
}
```

```ts
// Webhookを受けて単一エンティティだけ再検証
import { revalidateTag } from 'next/cache';

export async function POST(req: Request) {
  const { id } = await req.json();
  revalidateTag(`post:${id}`, { revalidate: 60 });
  return new Response('ok');
}
```

関連APIの使い分け
- revalidateTag: データ単位（タグ）でSWR再検証。横断的に共有データを更新したいとき。
- revalidatePath: ページ単位（URL）のレンダリング結果を無効化。特定ページを手っ取り早く更新したいとき。

落とし穴/注意
- タグ未付与のデータには効果なし（必ず取得側でタグを付ける）
- クライアント側からは直接呼べない（必要なら `router.refresh()` などで再描画を促す）
- `fetch` は GET のキャッシュが対象。POST/PUTの取得は対象外

#### updateTag()
**ServerActions専用の新API**であるupdateTagは、上記revalidateTagとは異なり「書き込み直後にその変更を即座に読み出せる」(read-your-writes) 一貫性を提供します。例えばユーザープロファイルを更新するアクション内でupdateTag('user-<id>')を呼び出すと、指定タグのキャッシュを即時に失効 (expire)させた上で最新データに更新し、同一リクエスト内で反映させることができます。これにより、フォーム送信直後にユーザー設定が画面に反映される、といった即時性の求められる機能に適しています。従来のrevalidateTagだと次のリクエストまで更新が反映されない場合がありましたが、updateTagは同一処理内でキャッシュを更新するため、ユーザーに一貫した最新の状態をすぐ提示できます。
```
'use server';
 
import { updateTag } from 'next/cache';
 
export async function updateUserProfile(userId: string, profile: Profile) {
  await db.users.update(userId, profile);
 
  // Expire cache and refresh immediately - user sees their changes right away
  updateTag(`user-${userId}`);
}
```

#### refresh()
もう一つの**Server Actions専用API**としてrefresh()が追加されました。この関数は**キャッシュされていないデータのみを更新するため**の、新しいサーバーアクション専用APIです。
例えば通知アイコンの未読数、ライブメトリック、ステータスインジケーターなど、ページ全体とは別に取得しておりキャッシュしていないデータを更新するケースで利用できます。refresh()を実行すると、クライアントサイドで提供されているrouter.refresh()と同様に、表示中ページを再レンダリングしますが、キャッシュ済みの部分には影響を与えません。そのため、ページの静的部分やキャッシュされたシェルは高速なまま、通知数やライブデータといった部分だけを更新することが可能です。

#### updateTag()とrefresh()の使い分け
「キャッシュされたデータ (タグ付き) を即時更新したい場合は updateTag、キャッシュしていない生データを更新したい場合は refresh」と使い分けることで、柔軟なデータ更新戦略が取れるようになります。
```
'use server';
 
import { refresh } from 'next/cache';
 
export async function markNotificationAsRead(notificationId: string) {
  // Update the notification in the database
  await db.notifications.markAsRead(notificationId);
 
  // Refresh the notification count displayed in the header
  // (which is fetched separately and not cached)
  refresh();
}
```
### React 19.2 と Canary 機能
Next.js 16のApp Router（appディレクトリ）は、Reactの最新Canary（開発版）リリースを採用しており、React 19.2の新機能にも対応しています。
React 19.2の詳細な内容はこちらです。ここは省略させてください（書き疲れたのは秘密で）
https://react.dev/blog/2025/10/01/react-19-2

### 削除された機能
過去のバージョンで非推奨と告知されていた機能・APIのいくつかは、Next.js 16で完全に削除されました。主な削除項目と代替・対応方法は下記の通りです。
- AMPサポートの削除: 旧来のAMP対応機能（例：useAmpフックやページコンフィグでamp: trueを指定する方法）は廃止されました。AMPビルド専用の仕組みはNext.js本体から取り除かれています。
- next lint コマンド削除: Next.js付属のnext lintコマンドは削除されました。代替としては、ESLint自体やFacebook製の静的解析ツールBiomeを直接使用することが推奨されます。なお、移行を支援するコード修正ツール（Codemod）として、npx @next/codemod@canary next-lint-to-eslint-cli .が提供されています。
- 開発インジケータ関連オプション削除: next.config.js内のdevIndicators設定で使用されていたappIsrStatus、buildActivity、buildActivityPositionといったオプションは削除されました。ビルドやISRの状況を示すインジケータ自体は残っていますが、これらをカスタマイズする設定項目がなくなった形です。
- Runtime Config (serverRuntimeConfig / publicRuntimeConfig) 削除: Next.jsで古くから提供されていたランタイム設定は廃止されました。serverRuntimeConfigおよびpublicRuntimeConfigに依存していたコードは、.envファイルや環境変数を用いた設定管理に移行する必要があります。
- Turbopack設定オプションの場所変更: next.config.js内でTurbopackを有効化するフラグの設定場所が変わりました。以前はexperimental.turbopack配下でしたが、Next.js 16ではトップレベルのturbopackプロパティに移動しています（既にTurbopackは実験段階を脱したため）。既存設定がある場合は書き換えが必要です。
- experimental.dynamicIO フラグ削除: これはCache Components以前に存在した試験的機能で、Next.js 16では**cacheComponents設定にリネーム**されたため旧フラグは削除されました。
- PPR関連設定の削除: 先述した通りexperimental.pprのグローバル設定や、ルートレベルでexport const experimental_ppr = ...と宣言する仕組みは削除されました。PPRの概念はCache Componentsに吸収されたためです。
- 自動scroll-behavior: smoothの削除: Next.jsはページ遷移時にスムーススクロールを自動適用していましたが、16ではそれが廃止されました。代替として、<html>タグにdata-scroll-behavior="smooth"属性を付与することで同様の効果を opt-in で得られます。
- unstable_rootParams()の削除: 特定の不安定APIであったunstable_rootParamsは削除されました（今後代替のAPIがマイナーバージョンで提供予定とのことです）。

以上の削除に該当する機能を使っている場合、Next.js 16にアップグレードするとビルドエラーやランタイムエラーが発生するため、事前にコード修正や設定変更が必要です。
https://nextjs.org/blog/next-16#removals

### 既定動作の変更
一部のデフォルト設定や動作が変更されています。変更内容は下記の通りです。
- デフォルトのバンドラ: 既述の通りTurbopackがデフォルトになりました。既存アプリを16にアップグレードすると自動的にTurbopackが使われます（Webpackを使い続けるには手動でフラグ指定する必要あり）。

- images.minimumCacheTTLのデフォルト延長: Next.jsの画像最適化で、キャッシュの最小TTL（Time To Live）が60秒から4時間(14400秒)に延長されました。これにより、キャッシュ制御ヘッダのない画像でも頻繁に再検証されず、デフォルトで長めにキャッシュされるようになります。

- images.imageSizesのデフォルト変更: <Image>コンポーネントで用いるレスポンシブ画像の生成サイズ一覧から、16ピクセル幅のエントリが削除されました（ほとんど使われていなかったため）。これにより不要な小サイズのsrcsetと追加リクエストが減り、画像最適化の効率が向上します。

- images.qualitiesのデフォルト変更: 画像クオリティ設定のデフォルトが[1, 2, ..., 100]（1から100まで全て）から**[75]（75のみ）**に簡素化されました。それに伴い、<Image>のqualityプロパティに任意の数値を指定しても、最も近い値（デフォルトでは常に75）に丸められます。開発者は必要に応じてimages.qualitiesを明示的に設定することになります。

- images.dangerouslyAllowLocalIPのデフォルト変更: セキュリティ向上のため、ローカルIPアドレスに対する画像最適化（例: 社内ネットワークの画像URLなど）はデフォルトでブロックされるようになりました。ローカルネットワーク内の画像を扱う場合は、この設定をtrueにしないと最適化が働かない点に注意してください（一般的な公開サイトでは通常不要な機能です）。

- images.maximumRedirectsのデフォルト設定: 画像最適化時に許容するリダイレクト回数が無制限から3回までに変更されました。これにより無限リダイレクトによるリソース浪費が防がれます（必要に応じて0に設定してリダイレクト禁止、またはさらに数値を増やすことも可能です）。

- ESLintプラグインのデフォルト設定: @next/eslint-plugin-nextがデフォルトでESLint Flat Config形式を使うようになりました。これは近くリリース予定のESLint v10で従来の設定形式が廃止されることを見据えた変更です。開発者側の対応は通常不要ですが、Flat Config形式への移行を念頭に置いておくと良いでしょう。

- プリフェッチキャッシュ動作: 前述の通りレイアウト重複排除とインクリメンタルプリフェッチが導入され、リンク事前取得の挙動が刷新されました。既存アプリでは自動的にこの新しい動作となり、開発者が手動で変更する必要はありません。

- revalidateTag()のシグネチャ: 上述した通り、revalidateTag(tag)のような引数一つの呼び出しは非推奨となり、デフォルトでは第二引数が必須の扱いに変わりました（16へのアップグレード時には対応が必要）。

- Turbopack使用時のBabel設定: Next.js 16では、Turbopack実行時にプロジェクトにBabel設定ファイルが存在すると自動的にBabelによる変換を有効化するようになりました。以前はBabel設定があるとエラーでビルドが停止していましたが、16ではWebpack同様にBabelを併用可能です（互換性向上）。

- ターミナル出力の刷新: 前述したログ改善に関連して、開発サーバー・ビルド時のターミナル出力のフォーマットが一新されています。より見やすいレイアウトや改善されたエラーメッセージ、パフォーマンス計測値の表示など、開発者が情報を把握しやすい出力に変わりました。

- 開発とビルドの出力ディレクトリ分離: next dev（開発）とnext build（プロダクションビルド）で別々の出力ディレクトリを使うようになりました。これにより、同じプロジェクトで開発サーバーを起動しながら別プロセスでビルドを実行するといった並行実行が可能になっています（以前は同じ.nextフォルダを使用していたため競合していました）。

- ロックファイル機構: 複数のnext devやnext buildを同一プロジェクトで誤って同時実行した場合の問題を防ぐため、ロックファイルによる排他制御が導入されました。これにより、意図しない競合実行を検知して警告することで、開発者のミスを減らします。

- Parallel Routesでのdefault.js必須化: App RouterのParallel Routes機能を使用している場合、各並行ルートのスロットに**default.jsファイルを明示的に配置**することが必須となりました。従来はスロットを埋めないとき省略可能でしたが、16ではビルド時に存在しないスロットがあるとエラーになります。空のスロットにはdefault.jsを作成し、中でnotFound()を呼び出すかnullを返す実装を入れることで対応できます。

- Sassローダーのアップデート: Next.js内蔵のSass処理が利用するsass-loaderがv16にバージョンアップされました。これにより最新のSass文法や新機能に対応しています。

https://nextjs.org/blog/next-16#behavior-changes
### 非推奨となった機能
- middleware.tsのファイル名: 前述の通り、middleware.tsは**proxy.tsへのリネーム**が推奨されています（現時点では動作しますが非推奨扱い）。将来のバージョンではmiddleware.tsはサポートされなくなる予定です。

- next/legacy/image コンポーネント: 旧版の画像コンポーネントであるnext/legacy/imageは非推奨となりました。**next/image**コンポーネントへの移行が必要です。新しいnext/imageはパフォーマンスや機能面で強化されているため、可能な限り早めに切り替えてください。

- images.domains 設定: next.configで特定のドメインの画像を最適化許可するためのimages.domains設定は非推奨になりました。代替として、**images.remotePatterns**設定を使用してください（より柔軟でセキュアなホスト許可設定が可能です）。

- revalidateTag()の引数省略: 先述の通り、revalidateTag('tag')のような単一引数呼び出しは非推奨です。revalidateTag(tag, profile)を指定するか、即時反映が必要な場合はupdateTag(tag)（Server Actions内）を使用するようにコードを更新してください。

https://nextjs.org/blog/next-16#deprecations

# 最後に
長らくお付き合いいただきありがとうございました。
この記事が誰かの助けになれば幸いです。