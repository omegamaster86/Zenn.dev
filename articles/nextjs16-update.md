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