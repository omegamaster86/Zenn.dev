---
title: "Next.js16公式とβ版について調査したんじゃ"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Next.js"]
published: false
# published_at: 2025-10-05 17:00
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
MCPに関してはもうみなさん知っていると思うので、できそうなことが



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

#### updateTag()


#### 

### 

### 