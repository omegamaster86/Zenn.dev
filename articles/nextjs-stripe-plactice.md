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
```:ProductList.tsx
'use client'
import { useCallback, useState, useReducer } from 'react'
import React from 'react'
import { Cart } from './cart'
import { CartSidebar } from './CartSidebar'
import { ProductCard } from './ProductCard'
import Category from './category'
import type { RamenItem, CartItem } from '@/app/types'
import { getStripe } from '@/app/lib/stripe'
import { createCheckoutSession } from '../server'

type CartAction = 
  | { type: 'ADD_ITEM'; item: RamenItem }
  | { type: 'REMOVE_ITEM'; itemId: string }
  | { type: 'UPDATE_QUANTITY'; itemId: string; quantity: number };

function cartReducer(state: CartItem[], action: CartAction): CartItem[] {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existingItemIndex = state.findIndex(cartItem => cartItem.item.id === action.item.id);
      if (existingItemIndex >= 0) {
        const updatedCart = [...state];
        updatedCart[existingItemIndex].quantity += 1;
        return updatedCart;
      }
      return [...state, { item: action.item, quantity: 1 }];
    }
    case 'REMOVE_ITEM':
      return state.filter(cartItem => cartItem.item.id !== action.itemId);
    case 'UPDATE_QUANTITY':
      return state.map(cartItem =>
        cartItem.item.id === action.itemId
          ? { ...cartItem, quantity: action.quantity }
          : cartItem
      );
    default:
      return state;
  }
}

export const ProductList = ({ initialProducts }: { initialProducts: RamenItem[] }) => {
  const [cart, dispatch] = useReducer(cartReducer, []);
  const [selectedCategory, setSelectedCategory] = useState<RamenItem['category']>('all');
  const [isCartOpen, setIsCartOpen] = useState(false);

  const filteredItems = selectedCategory === 'all' 
    ? initialProducts 
    : initialProducts.filter(item => item.category === selectedCategory);

  const addToCart = useCallback((item: RamenItem) => {
    dispatch({ type: 'ADD_ITEM', item });
    setIsCartOpen(true);
  }, [dispatch]);
  
  const removeFromCart = useCallback((itemId: string) => {
    dispatch({ type: 'REMOVE_ITEM', itemId });
  }, [dispatch]);

  const updateQuantity = useCallback((itemId: string, quantity: number) => {
    if (quantity < 1) return;
    dispatch({ type: 'UPDATE_QUANTITY', itemId, quantity });
  }, [dispatch]);

  const totalPrice = cart.reduce((total, cartItem) => 
    total + (cartItem.item.price * cartItem.quantity), 0
  );

  const handleCheckout = async () => {
    try {
      const { sessionId } = await createCheckoutSession(cart, totalPrice);
      
      const stripe = await getStripe();
      if (!stripe) {
        throw new Error('Stripeの初期化に失敗しました');
      }

      await stripe.redirectToCheckout({ sessionId });
    } catch (err) {
      console.error('Error:', err);
      alert('決済処理中にエラーが発生しました');
    }
  };
  
  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-8 text-center">らーめん屋 匠</h1>
      
      <Cart 
        cart={cart} 
        isCartOpen={isCartOpen} 
        setIsCartOpen={setIsCartOpen} 
      />

      <CartSidebar
        cart={cart}
        isCartOpen={isCartOpen}
        setIsCartOpen={setIsCartOpen}
        updateQuantity={updateQuantity}
        removeFromCart={removeFromCart}
        totalPrice={totalPrice}
        handleCheckout={handleCheckout}
      />
      
      <Category 
        selectedCategory={selectedCategory} 
        setSelectedCategory={setSelectedCategory}
      />
      
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {filteredItems.map((item) => (
          <ProductCard
            key={item.id}
            item={item}
            onAddToCart={addToCart}
          />
        ))}
      </div>
      
      {filteredItems.length === 0 && (
        <p className="text-center text-gray-500 mt-8">該当する商品がありません</p>
      )}
    </div>
  );
} 
```
`useReducer`を使用したことがなかったので、使ってみました。
違う書き方もあるとは思いますが、今回は練習ということで見逃していただきたいです。
```:server.tsx
export async function createCheckoutSession(cart: CartItem[], totalPrice: number) {
  try {
    const stripe = new Stripe(process.env.NEXT_PUBLIC_STRIPE_SECRET_KEY!, {
      apiVersion: '2025-01-27.acacia',
      maxNetworkRetries: 3
    });

    const lineItems = cart.map(cartItem => ({
      price_data: {
        currency: 'jpy',
        product_data: {
          name: cartItem.item.name,
          images: [cartItem.item.imageUrl],
        },
        unit_amount: cartItem.item.price,
      },
      quantity: cartItem.quantity
    }));
    
    const session = await stripe.checkout.sessions.create({
      payment_method_types: ['card'],
      line_items: lineItems,
      mode: 'payment',
      success_url: `${process.env.BASE_URL}/dashboard/success?session_id={CHECKOUT_SESSION_ID}`,
      cancel_url: `${process.env.BASE_URL}/dashboard/shopping`,
      metadata: {
        totalAmount: totalPrice.toString()
      }
    });

    return { sessionId: session.id };
  } catch (error) {
    console.error('Error:', error);
    throw new Error('決済処理中にエラーが発生しました');
  }
} 
```
### ショッピングカートデータの変換を`lineItems`で行います。
price_data内では、
currency: 通貨を 'jpy'（日本円）に固定しています。
product_data: 商品の情報として、名前（cartItem.item.name）と画像（cartItem.item.imageUrl）を設定します。
unit_amount: 単価（価格）を設定。Stripe は金額を最小単位（円なら1円単位）で扱います。
quantity：商品の購入数量が設定されます。
### Checkout セッションの作成
セッション作成 API 呼び出し
Stripe の checkout.sessions.create メソッドを利用して、決済セッションを生成しています。ここで、各パラメータの意味は次の通りです。
各パラメータの詳細
payment_method_types: 利用可能な支払い方法を指定しており、ここではクレジットカードのみを許可しています。
line_items: 先ほど作成した、各商品の詳細を含む配列です。
mode:'payment'とすることで、一回限りの支払いセッションであることを示しています（サブスクリプションなどでは異なるモードを指定します）。
success_urlとcancel_url:
決済完了時やキャンセル時にリダイレクトするURLです。環境変数 process.env.BASE_URL を使って基本URLを指定し、各シーンに応じたパスを設定しています。
特に success_url では {CHECKOUT_SESSION_ID} のプレースホルダを用いて、セッションIDをクエリパラメータとして渡せるようにしています。
metadata:セッションに追加情報を付加できる仕組みです。ここでは、合計金額（totalPrice）を文字列に変換して保存しています。これにより、後で決済情報と紐付けた追加データとして利用できます。
### 戻り値
作成された Checkout セッションのIDを返すことで、クライアント側がそのIDを使って決済プロセスを進められるようにします。

```:ProductCard.tsx
'use client'

import React from 'react'
import Image from 'next/image'
import type { RamenItem } from '@/app/types'

interface ProductCardProps {
  item: RamenItem;
  onAddToCart: (item: RamenItem) => void;
}

export const ProductCard = ({ item, onAddToCart }: ProductCardProps) => {
  return (
    <div className="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow duration-300">
      <div className="h-48 relative">
        <Image
          src={item.imageUrl}
          alt={item.name}
          fill
          style={{ objectFit: 'cover' }}
          priority
        />
      </div>
      <div className="p-4">
        <h3 className="text-xl font-semibold mb-2">{item.name}</h3>
        <p className="text-red-600 font-bold mb-2">¥{item.price.toLocaleString()}</p>
        <div className="mb-4">
          <p className="text-sm text-gray-600 font-medium">アレルギー:</p>
          <div className="flex flex-wrap gap-1 mt-1">
            {item.allergies.map((allergy, index) => (
              <span key={index} className="inline-block bg-yellow-100 text-yellow-800 text-xs px-2 py-1 rounded">
                {allergy}
              </span>
            ))}
          </div>
        </div>
        <button 
          onClick={() => onAddToCart(item)}
          className="w-full bg-red-600 hover:bg-red-700 text-white py-2 rounded-md transition-colors duration-300"
        >
          カートに追加
        </button>
      </div>
    </div>
  );
}; 
```
ここではstripeにある情報を取得して表示しています。
ここで`Image`を有効にするのは下記のファイルに`Image`の箇所を追記してください。
```:nextConfig.tsx
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  experimental: {
    ppr: true,
    newDevOverlay: true
  },
  images: {
    domains: ['files.stripe.com'],
  },
};

export default nextConfig;
```
```:lib/stripe.ts
import { loadStripe } from '@stripe/stripe-js';

let stripePromise: Promise<any> | null = null;

export const getStripe = () => {
  if (!stripePromise) {
    stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!);
  }
  return stripePromise;
}; 
```
### Stripe ライブラリのインポート
loadStripe を @stripe/stripe-js からインポートし、Stripe の公開可能キーを使ってライブラリをロードする関数を利用できるようにしています。
### キャッシュ変数の用意
stripePromise という変数を定義し、初期値を null に設定。ここに一度ロードした Stripe の Promise を保存して、再ロードを防いでいます。
### getStripe 関数の定義
### 初回呼び出し時
stripePromise がまだ null の場合、loadStripe に環境変数から取得した公開可能キーを渡して Stripe ライブラリのロードを開始し、その Promise を stripePromise に格納します。
### 以降の呼び出し時
すでに stripePromise に Promise が設定されている場合、再度ロードするのではなく、キャッシュされた Promise を返します。
この仕組みにより、Stripe のライブラリが一度だけロードされ、複数回呼び出しても同じインスタンスが再利用されるようになっています。これにより、無駄なネットワークリクエストを避け、パフォーマンスを向上させることができます。

下記は特記事項がないため省略
```:Cart.tsx
import React from 'react'
import type { CartItem } from '@/app/types'

interface CartProps {
  cart: CartItem[];
  isCartOpen: boolean;
  setIsCartOpen: (isOpen: boolean) => void;
}

export const Cart = ({ cart, isCartOpen, setIsCartOpen }: CartProps) => {
  return (
    <button 
      onClick={() => setIsCartOpen(!isCartOpen)}
      className="fixed top-20 right-4 bg-red-600 text-white p-2 rounded-full shadow-lg hover:bg-red-700 transition-colors duration-300"
    >
      <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
      </svg>
      {cart.length > 0 && (
        <span className="absolute -top-1 -right-1 bg-yellow-500 text-xs w-5 h-5 flex items-center justify-center rounded-full">
          {cart.length}
        </span>
      )}
    </button>
  )
}

export default Cart
```
```:button.tsx
import { ButtonHTMLAttributes } from 'react';
import { twMerge } from 'tailwind-merge';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary';
  size?: 'md';
  isActive?: boolean;
}

export const Button = ({
  children,
  className,
  variant = 'primary',
  size = 'md',
  isActive = false,
  ...props
}: ButtonProps) => {
  const baseStyles = 'rounded-full transition-colors duration-300';
  
  const variants = {
    primary: 'bg-red-600 text-white hover:bg-red-700',
    secondary: 'bg-gray-200 hover:bg-gray-300',
    outline: 'border border-gray-300 hover:bg-gray-100'
  };

  const sizes = {
    sm: 'px-3 py-1 text-sm',
    md: 'px-4 py-2',
    lg: 'px-6 py-3 text-lg'
  };

  const activeStyles = isActive ? variants.primary : variants.secondary;
  const variantStyles = isActive !== undefined ? activeStyles : variants[variant];
  
  return (
    <button
      className={twMerge(
        baseStyles,
        variantStyles,
        sizes[size],
        className
      )}
      {...props}
    >
      {children}
    </button>
  );
}; 
```
```:Category.tsx
import React from 'react'
import type { RamenItem } from '@/app/types'
import { Button } from '../_components/button'

type CategoryProps = {
  selectedCategory: RamenItem['category'];
  setSelectedCategory: React.Dispatch<React.SetStateAction<RamenItem['category']>>;
}

const Category = ({ selectedCategory, setSelectedCategory }: CategoryProps) => {
  return (
    <div className="flex justify-center mb-8 space-x-4">
      <Button
        onClick={() => setSelectedCategory('all')}
        isActive={selectedCategory === 'all'}
      >
        すべて
      </Button>
      <Button
        onClick={() => setSelectedCategory('shoyu')}
        isActive={selectedCategory === 'shoyu'}
      >
        醤油ラーメン
      </Button>
      <Button
        onClick={() => setSelectedCategory('miso')}
        isActive={selectedCategory === 'miso'}
      >
        味噌ラーメン
      </Button>
    </div>
  )
}

export default Category
```
```:CartSidebar.tsx
'use client'

import React from 'react'
import Image from 'next/image'
import type { CartItem } from '@/app/types'

interface CartSidebarProps {
  cart: CartItem[];
  isCartOpen: boolean;
  setIsCartOpen: (isOpen: boolean) => void;
  updateQuantity: (itemId: string, quantity: number) => void;
  removeFromCart: (itemId: string) => void;
  totalPrice: number;
  handleCheckout: () => Promise<void>;
}

export const CartSidebar = ({
  cart,
  isCartOpen,
  setIsCartOpen,
  updateQuantity,
  removeFromCart,
  totalPrice,
  handleCheckout
}: CartSidebarProps) => {
  if (!isCartOpen) return null;

  return (
    <div className="fixed inset-0 z-50 overflow-hidden">
      <div className="absolute inset-0 bg-opacity-50" onClick={() => setIsCartOpen(false)}></div>
      <div className="absolute right-0 top-0 h-full w-full max-w-md bg-white shadow-xl transform transition-all">
        <div className="p-4 h-full flex flex-col">
          <div className="flex justify-between items-center mb-4">
            <h2 className="text-xl font-bold">カート</h2>
            <button onClick={() => setIsCartOpen(false)} className="text-gray-500 hover:text-gray-700">
              <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
              </svg>
            </button>
          </div>
          
          {cart.length === 0 ? (
            <div className="flex-1 flex items-center justify-center">
              <p className="text-gray-500">カートは空です</p>
            </div>
          ) : (
            <>
              <div className="flex-1 overflow-y-auto">
                {cart.map(cartItem => (
                  <div key={cartItem.item.id} className="flex items-center py-4 border-b">
                    <div className="w-16 h-16 relative mr-4">
                      <Image
                        src={cartItem.item.imageUrl}
                        alt={cartItem.item.name}
                        fill
                        style={{ objectFit: 'cover' }}
                        className="rounded"
                      />
                    </div>
                    <div className="flex-1">
                      <h3 className="font-medium">{cartItem.item.name}</h3>
                      <p className="text-red-600">¥{cartItem.item.price}</p>
                    </div>
                    <div className="flex items-center">
                      <button 
                        onClick={() => updateQuantity(cartItem.item.id, cartItem.quantity - 1)}
                        className="w-8 h-8 flex items-center justify-center bg-gray-200 rounded-l"
                      >
                        -
                      </button>
                      <span className="w-8 h-8 flex items-center justify-center bg-gray-100">
                        {cartItem.quantity}
                      </span>
                      <button 
                        onClick={() => updateQuantity(cartItem.item.id, cartItem.quantity + 1)}
                        className="w-8 h-8 flex items-center justify-center bg-gray-200 rounded-r"
                      >
                        +
                      </button>
                      <button 
                        onClick={() => removeFromCart(cartItem.item.id)}
                        className="ml-2 text-red-500"
                      >
                        <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                        </svg>
                      </button>
                    </div>
                  </div>
                ))}
              </div>
              <div className="pt-4 border-t">
                <div className="flex justify-end mb-4">
                  <span className="font-bold">合計:</span>
                  <span className="font-bold">¥{totalPrice}</span>
                </div>
                <button 
                  onClick={handleCheckout}
                  className="w-full bg-red-600 hover:bg-red-700 text-white py-3 rounded-md"
                >
                  注文する
                </button>
              </div>
            </>
          )}
        </div>
      </div>
    </div>
  );
}; 
```
