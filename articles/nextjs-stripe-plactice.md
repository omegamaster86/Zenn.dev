---
title: "Next.jsとstripeで実装したいんじゃ"
emoji: "💰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react","Next.js","TypeScript","stripe",]
published: false
# published_at: 2025-02-17 10:00
---

# 初めに
さて皆さん、今頃ワイルズをやっていると思いますがクリア出来ましたか？
私はPS5を買ってもいないのでやっていませんが、ワイルズの感想をお待ちしています！
さて本題ですが、発注ナビというサイトにてこんな案件がありました。
「学校の食堂で券売機ではなく、ネットから注文できるようにしたい」
イメージで言うと飲食店では最近タブレットで注文すると思いますが、それに学生ごとのログイン機能がついたものだと仮定します。
今回は学食での使用イメージですが、飲食店全般に使用できそうですね〜
特にラーメン屋は券売機が多いので、新紙幣になると戸惑いが多かったのは記憶に新しいかと思います。
ただ「飲食店を使用するのにアカウント登録する」という違和感はありますが、目を瞑りましょう！
ps。あとお金があるところでないと手数料的に難しいですね…（今更感…）

# 画面図
### 商品一覧ページ
![](/images/nextjs-stripe-plactice/image1.png)
### 「カートに入れる」ボタン押下時
![](/images/nextjs-stripe-plactice/image2.png)
### 支払い画面
![](/images/nextjs-stripe-plactice/image3.png)

# 実際のコード
```:types/index.ts
export interface RamenItem {
  id: string;
  name: string;
  price: number;
  category: 'all' | 'shoyu' | 'miso';
  allergies: string[];
  imageUrl: any;
  stripeProductId: string;
  stripePriceId: string;
}

export interface CartItem {
  item: RamenItem;
  quantity: number;
} 
```
型の説明は省きますね〜

```:page.tsx
import { ProductList } from './_components/ProductList';
import { getProducts } from './server';

export default async function Page() {
  const products = await getProducts();
  return <ProductList initialProducts={products} />;
}
```
`server`から`getProducts`を呼び出しています。
ファイル先頭のu`se server`ディレクティブにより、ファイル内のすべての関数がサーバーアクションとしてマークされ、結果として下記のメリットがあります。
- セキュリティの向上
サーバー上でデータフェッチやビジネスロジックを処理するため、クライアント側に機密情報（APIキーやデータベース接続情報など）が露出しないため
- SEO（検索エンジン最適化）への効果
サーバーサイドレンダリング（SSR）では、初期HTMLにコンテンツが含まれるため、検索エンジンがページの内容を容易にクロールでき、SEO効果がある
- 効率的なデータ処理とキャッシュ戦略
サーバーサイドでデータを一括して取得・処理できるため、クライアント側で複数回のリクエストを発行する必要がなく、キャッシュやデータ集約の戦略を柔軟に適用できます。

```:server.tsx
export async function getProducts(): Promise<RamenItem[]> {
  const stripe = new Stripe(process.env.NEXT_PUBLIC_STRIPE_SECRET_KEY!, {
    apiVersion: '2025-01-27.acacia',
    maxNetworkRetries: 3
  });

  try {
    const products = await stripe.products.list({
      expand: ['data.default_price'],
      active: true,
    });

    return products.data.map(product => ({
      id: product.id,
      name: product.name,
      price: (product.default_price as Stripe.Price)?.unit_amount ?? 0,
      category: product.metadata?.category as RamenItem['category'] || 'all',
      allergies: product.metadata?.allergies ? 
        product.metadata.allergies.split(',').map(a => a.trim()) : [],
      imageUrl: product.images?.[0] || '',
      stripeProductId: product.id,
      stripePriceId: (product.default_price as Stripe.Price)?.id ?? ''
    }));
  } catch (error) {
    console.error('Stripe products fetch error:', error);
    return [];
  }
}
```
環境変数からの API キー取得
process.env.NEXT_PUBLIC_STRIPE_SECRET_KEY! により、環境変数から秘密鍵を取得しています。! は TypeScript の non-null assertion で、必ず値が存在すると宣言しています。
:::message
Stripe API の初期化をする理由

認証とセキュリティの確保
初期化時に環境変数から取得した API キーを利用して、Stripe サーバーに対する認証が行われます。これにより、不正なリクエストやアクセスからシステムを保護できます。

API バージョンの固定
初期化オプションで API バージョン（例: '2025-01-27.acacia'）を指定することで、将来的な仕様変更や互換性の問題を回避し、常に予測可能な挙動でStripeの機能を利用できます。

ネットワークエラーに対する耐性
オプションで maxNetworkRetries を設定することにより、通信中の一時的なエラー発生時に自動的にリトライを行い、API 呼び出しの信頼性を高めています。

クライアントオブジェクトの生成
初期化によって Stripe クライアントオブジェクトが作成されるため、その後はこのオブジェクトを介して一貫した方法でStripeの各種API（例えば、stripe.products.list）を呼び出すことが可能になります。これにより、コード全体の保守性と可読性が向上します。
:::

Stripe API の初期化しない場合、下記のような不具合が発生する可能性があります。
:::message alert
API バージョンの不整合
初期化時に API バージョンを指定していないと、デフォルトのバージョンが使用されることになります。これにより、将来的な仕様変更や新機能との互換性に問題が生じる可能性があります。

ネットワークのリトライ設定の欠如
maxNetworkRetries などの設定が行われないため、一時的なネットワーク障害が発生した際に自動的な再試行が行われず、エラーがそのまま伝播します。

クライアントオブジェクトの不在
Stripe API にアクセスするためのクライアントオブジェクトが生成されないため、Stripe の各種 API メソッドを呼び出すことができなくなります。
:::
設定オプション
apiVersion: 使用する Stripe API のバージョンを指定します。
maxNetworkRetries: ネットワークエラー時にリトライする回数を指定しています。
expand オプション：'data.default_price' を指定することで、各商品の default_price オブジェクトも一緒に取得し、後の処理で直接参照できるようにしています。
active フィルタ：active: true とすることで、現在有効な商品だけを対象にしています。
