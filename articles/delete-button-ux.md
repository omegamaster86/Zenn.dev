---
title: "よりよい削除ボタンのUI"
emoji: "🆑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["React","Next","HTML"]
published: false
# published_at: 2025-06-07 18:30
---

# はじめに
どうも！毎回Zennの記事タイトルの上にある絵文字の選択に困っているオメガマスターです。
なんかランダムで選んでくれて良いのになと思っています。
今回は下記の記事を見てこういう削除ボタン良いなと思いました。
https://emilkowal.ski/ui/building-a-hold-to-delete-component
ちなみに皆さんの会社ではどのような削除ボタンを実装していますか？
- 削除ボタンをおいて、クリックされたらアラートが表示され、yesをクリックすると削除

しか弊社では実装していない気がします。それでも良いかなと思うのですが、慣れてきたり、急いだりしていると確認も適当になって削除しちゃうこともあると思います。そんな中このUIならオシャンティかつ、間違えることもないのでUXよさそうな気がしています。

# 実装
サイトにもコードはありますが、tailwindを普段から使用しているので、変更していきたいと思います。
完成系のコードは下記です。

```:type.ts
export interface Post {
  id: number;
  title: string;
  content: string;
  created_at: string;
  updated_at: string;
} 
```
型の説明は特に不要だと思うので割愛します。
```:server.tsx
import { Post } from '@/app/types';

const API_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:3001';

export async function getPosts(): Promise<Post[]> {
  try {
    const response = await fetch(`${API_URL}/api/v1/posts`, {
      cache: 'no-store',
    });
    
    if (!response.ok) {
      throw new Error('投稿の取得に失敗しました');
    }
    
    return await response.json();
  } catch (error) {
    console.error('Posts fetch error:', error);
    return [];
  }
} 
```
Promise<Post[]> は、関数 getPosts() が 非同期で Post[] 型（投稿の配列）を返すことを示す型注釈 です。
なので上記のコードから`: Promise<Post[]>`を削除しても問題ありません。（型が効かなくなるので、問題といえば問題かもしれませんが…）promiseに関して「あれ？」と思った方はこちらを見てください。
https://qiita.com/cheez921/items/41b744e4e002b966391a
https://zenn.dev/peter_norio/articles/289927bab5a4d3

`cache: 'no-store'`はfetch に渡しているオプションで、ブラウザやNext.jsのキャッシュを使わず、常にサーバーから新鮮なデータを取得する設定です。
他の設定はこちらです。
| オプション名             | 説明                                                      |
| ------------------ | ------------------------------------------------------- |
| `default`        | ブラウザやNext.jsの通常のキャッシュポリシーに従う（状況によって使ったり使わなかったり）         |
| `no-store`       | **キャッシュを一切使わず、毎回必ずサーバーから新しいデータを取得する**                   |
| `reload`         | サーバーからデータを取得し、キャッシュを**上書き**する。キャッシュは**使わない**            |
| `no-cache`       | **キャッシュがあっても必ずサーバーに確認する**。条件付きGETなどで最新であればキャッシュを使うこともある |
| `force-cache`    | サーバーに問い合わせず、**常にキャッシュを使う**（キャッシュがなければサーバーに問い合わせる）       |
| `only-if-cached` | **キャッシュがあるときだけ返す**。なければエラーになる（ブラウザでしか使えないことが多い）         |

なんとなく'`no-store`と`reload`が同じように見えますが、実際には違いがあります。
 | オプション        | キャッシュから読む？ | サーバーに問い合わせる？ | キャッシュに保存する？   |
| ------------ | ---------- | ------------ | ------------- |
| `no-store` | ❌ 一切使わない   | ✅ 必ず問い合わせる   | ❌ キャッシュに保存しない |
| `reload`   | ❌ 一切使わない   | ✅ 必ず問い合わせる   | ✅ キャッシュに保存する  |

`no-store`は管理画面やAPI通信など、絶対に最新データを使いたい場面やセキュリティ的にキャッシュしてはいけない場合（例：個人情報）に使用します。
`reload`は初回はサーバーから最新を取得したいが、次回以降はキャッシュを使いたい場面やキャッシュを明示的に更新したいとき（リロードボタン的な動作）に使用します。
```:page.tsx
import { getPosts } from './server';
import ClientHome from '@/app/client-home';

export default async function Home() {
  const initialPosts = await getPosts();
  
  return <ClientHome initialPosts={initialPosts} />;
}
```
`const initialPosts = await getPosts();`の`getPosts()`にカーソルを当てると`promise<Post[]>`があるので、型が渡っています。
```:delete-button.tsx
"use client";
import { useRef } from "react";

interface DeleteButtonProps {
  onDelete?: () => void;
}

const DeleteButton = ({ onDelete }: DeleteButtonProps) => {
  const overlayRef = useRef<HTMLDivElement>(null);
  const timeoutRef = useRef<NodeJS.Timeout | null>(null);

  const handleMouseDown = () => {
    
    if (overlayRef.current) {
      overlayRef.current.style.clipPath = 'inset(0px 0px 0px 0px)';
      overlayRef.current.style.transition = 'clip-path 2s linear';
    }

    timeoutRef.current = setTimeout(() => {
      onDelete?.();
    }, 2000);
  };

  const handleMouseUp = () => {
    
    if (overlayRef.current) {
      overlayRef.current.style.clipPath = 'inset(0px 100% 0px 0px)';
      overlayRef.current.style.transition = 'clip-path 200ms ease-out';
    }
    
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
      timeoutRef.current = null;
    }
  };

  const handleMouseLeave = () => {
    
    if (overlayRef.current) {
      overlayRef.current.style.clipPath = 'inset(0px 100% 0px 0px)';
      overlayRef.current.style.transition = 'clip-path 200ms ease-out';
    }
    
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
      timeoutRef.current = null;
    }
  };

  return (
    <button 
      className="relative flex h-10 items-center gap-2 rounded-full bg-stone-100 px-6 font-medium text-stone-800 select-none transition-transform duration-150 ease-out"
      onMouseDown={handleMouseDown}
      onMouseUp={handleMouseUp}
      onMouseLeave={handleMouseLeave}
      onTouchStart={handleMouseDown}
      onTouchEnd={handleMouseUp}
    >
      <div 
        ref={overlayRef}
        aria-hidden="true" 
        className="absolute inset-0 flex items-center justify-center gap-2 rounded-full bg-red-100 text-red-500"
        style={{
          clipPath: 'inset(0px 100% 0px 0px)',
          transition: 'clip-path 200ms ease-out'
        }}
      >
        <svg height="16" strokeLinejoin="round" viewBox="0 0 16 16" width="16">
          <path
            fillRule="evenodd"
            clipRule="evenodd"
            d="M6.75 2.75C6.75 2.05964 7.30964 1.5 8 1.5C8.69036 1.5 9.25 2.05964 9.25 2.75V3H6.75V2.75ZM5.25 3V2.75C5.25 1.23122 6.48122 0 8 0C9.51878 0 10.75 1.23122 10.75 2.75V3H12.9201H14.25H15V4.5H14.25H13.8846L13.1776 13.6917C13.0774 14.9942 11.9913 16 10.6849 16H5.31508C4.00874 16 2.92263 14.9942 2.82244 13.6917L2.11538 4.5H1.75H1V3H1.75H3.07988H5.25ZM4.31802 13.5767L3.61982 4.5H12.3802L11.682 13.5767C11.6419 14.0977 11.2075 14.5 10.6849 14.5H5.31508C4.79254 14.5 4.3581 14.0977 4.31802 13.5767Z"
            fill="currentColor"
          />
        </svg>
        Hold to Delete
      </div>
      <svg height="16" strokeLinejoin="round" viewBox="0 0 16 16" width="16">
        <path
          fillRule="evenodd"
          clipRule="evenodd"
          d="M6.75 2.75C6.75 2.05964 7.30964 1.5 8 1.5C8.69036 1.5 9.25 2.05964 9.25 2.75V3H6.75V2.75ZM5.25 3V2.75C5.25 1.23122 6.48122 0 8 0C9.51878 0 10.75 1.23122 10.75 2.75V3H12.9201H14.25H15V4.5H14.25H13.8846L13.1776 13.6917C13.0774 14.9942 11.9913 16 10.6849 16H5.31508C4.00874 16 2.92263 14.9942 2.82244 13.6917L2.11538 4.5H1.75H1V3H1.75H3.07988H5.25ZM4.31802 13.5767L3.61982 4.5H12.3802L11.682 13.5767C11.6419 14.0977 11.2075 14.5 10.6849 14.5H5.31508C4.79254 14.5 4.3581 14.0977 4.31802 13.5767Z"
          fill="currentColor"
        />
      </svg>
      Hold to Delete
    </button>
  );
}

export default DeleteButton
```

# おまけ
なんとなくcursorとclaudeで出力結果かわるのかな〜と思い、実験してみたら、結構違う出力になりました…
どちらもclaude-4-sonnetを使用しています。
cursorでの出力
```
"use client";

import { useState, useRef } from 'react';

const DeleteButton = () => {
  const [isPressed, setIsPressed] = useState(false);
  const [progress, setProgress] = useState(0);
  const timeoutRef = useRef<NodeJS.Timeout | null>(null);
  const intervalRef = useRef<NodeJS.Timeout | null>(null);

  const handleMouseDown = () => {
    setIsPressed(true);
    setProgress(0);
    
    // プログレスアニメーション
    intervalRef.current = setInterval(() => {
      setProgress(prev => {
        if (prev >= 100) {
          handleDelete();
          return 100;
        }
        return prev + 2; // 2秒で100%に到達
      });
    }, 40); // 40msごとに更新（滑らかなアニメーション）
  };

  const handleMouseUp = () => {
    setIsPressed(false);
    setProgress(0);
    if (timeoutRef.current) clearTimeout(timeoutRef.current);
    if (intervalRef.current) clearInterval(intervalRef.current);
  };

  const handleDelete = () => {
    console.log('削除実行');
    // ここに削除ロジックを実装
    setIsPressed(false);
    setProgress(0);
    if (timeoutRef.current) clearTimeout(timeoutRef.current);
    if (intervalRef.current) clearInterval(intervalRef.current);
  };

  return (
    <button 
      className="relative flex h-10 items-center gap-2 rounded-full bg-stone-100 px-6 font-medium text-stone-800 select-none transition-transform duration-150 ease-out active:scale-[0.97] overflow-hidden"
      onMouseDown={handleMouseDown}
      onMouseUp={handleMouseUp}
      onMouseLeave={handleMouseUp}
      onTouchStart={handleMouseDown}
      onTouchEnd={handleMouseUp}
    >
      {/* 赤いオーバーレイ */}
      <div 
        className="absolute inset-0 flex items-center justify-center gap-2 rounded-full bg-red-100 text-red-500 transition-all duration-200 ease-out"
        style={{
          clipPath: `inset(0px ${100 - progress}% 0px 0px)`,
          opacity: isPressed ? 1 : 0
        }}
      >
        <svg height="16" strokeLinejoin="round" viewBox="0 0 16 16" width="16">
          <path
            fillRule="evenodd"
            clipRule="evenodd"
            d="M6.75 2.75C6.75 2.05964 7.30964 1.5 8 1.5C8.69036 1.5 9.25 2.05964 9.25 2.75V3H6.75V2.75ZM5.25 3V2.75C5.25 1.23122 6.48122 0 8 0C9.51878 0 10.75 1.23122 10.75 2.75V3H12.9201H14.25H15V4.5H14.25H13.8846L13.1776 13.6917C13.0774 14.9942 11.9913 16 10.6849 16H5.31508C4.00874 16 2.92263 14.9942 2.82244 13.6917L2.11538 4.5H1.75H1V3H1.75H3.07988H5.25ZM4.31802 13.5767L3.61982 4.5H12.3802L11.682 13.5767C11.6419 14.0977 11.2075 14.5 10.6849 14.5H5.31508C4.79254 14.5 4.3581 14.0977 4.31802 13.5767Z"
            fill="currentColor"
          />
        </svg>
        Hold to Delete
      </div>

      {/* 通常の表示 */}
      <svg height="16" strokeLinejoin="round" viewBox="0 0 16 16" width="16">
        <path
          fillRule="evenodd"
          clipRule="evenodd"
          d="M6.75 2.75C6.75 2.05964 7.30964 1.5 8 1.5C8.69036 1.5 9.25 2.05964 9.25 2.75V3H6.75V2.75ZM5.25 3V2.75C5.25 1.23122 6.48122 0 8 0C9.51878 0 10.75 1.23122 10.75 2.75V3H12.9201H14.25H15V4.5H14.25H13.8846L13.1776 13.6917C13.0774 14.9942 11.9913 16 10.6849 16H5.31508C4.00874 16 2.92263 14.9942 2.82244 13.6917L2.11538 4.5H1.75H1V3H1.75H3.07988H5.25ZM4.31802 13.5767L3.61982 4.5H12.3802L11.682 13.5767C11.6419 14.0977 11.2075 14.5 10.6849 14.5H5.31508C4.79254 14.5 4.3581 14.0977 4.31802 13.5767Z"
          fill="currentColor"
        />
      </svg>
      Hold to Delete
    </button>
  );
}

export default DeleteButton
```

claudeでの出力
```
"use client";
import { useState, useRef } from "react";

export default function ClipPathButton() {
  const [isPressed, setIsPressed] = useState(false);
  const overlayRef = useRef(null);

  const handleMouseDown = () => {
    setIsPressed(true);
    if (overlayRef.current) {
      overlayRef.current.style.clipPath = 'inset(0px 0px 0px 0px)';
      overlayRef.current.style.transition = 'clip-path 2s linear';
    }
  };

  const handleMouseUp = () => {
    setIsPressed(false);
    if (overlayRef.current) {
      overlayRef.current.style.clipPath = 'inset(0px 100% 0px 0px)';
      overlayRef.current.style.transition = 'clip-path 200ms ease-out';
    }
  };

  const handleMouseLeave = () => {
    setIsPressed(false);
    if (overlayRef.current) {
      overlayRef.current.style.clipPath = 'inset(0px 100% 0px 0px)';
      overlayRef.current.style.transition = 'clip-path 200ms ease-out';
    }
  };

  return (
    <button 
      className={`relative flex h-10 items-center gap-2 rounded-full bg-stone-100 px-6 font-medium text-stone-800 select-none transition-transform duration-150 ease-out ${isPressed ? 'scale-[0.97]' : ''}`}
      onMouseDown={handleMouseDown}
      onMouseUp={handleMouseUp}
      onMouseLeave={handleMouseLeave}
      onTouchStart={handleMouseDown}
      onTouchEnd={handleMouseUp}
    >
      <div 
        ref={overlayRef}
        aria-hidden="true" 
        className="absolute inset-0 flex items-center justify-center gap-2 rounded-full bg-red-100 text-red-500"
        style={{
          clipPath: 'inset(0px 100% 0px 0px)',
          transition: 'clip-path 200ms ease-out'
        }}
      >
        <svg height="16" strokeLinejoin="round" viewBox="0 0 16 16" width="16">
          <path
            fillRule="evenodd"
            clipRule="evenodd"
            d="M6.75 2.75C6.75 2.05964 7.30964 1.5 8 1.5C8.69036 1.5 9.25 2.05964 9.25 2.75V3H6.75V2.75ZM5.25 3V2.75C5.25 1.23122 6.48122 0 8 0C9.51878 0 10.75 1.23122 10.75 2.75V3H12.9201H14.25H15V4.5H14.25H13.8846L13.1776 13.6917C13.0774 14.9942 11.9913 16 10.6849 16H5.31508C4.00874 16 2.92263 14.9942 2.82244 13.6917L2.11538 4.5H1.75H1V3H1.75H3.07988H5.25ZM4.31802 13.5767L3.61982 4.5H12.3802L11.682 13.5767C11.6419 14.0977 11.2075 14.5 10.6849 14.5H5.31508C4.79254 14.5 4.3581 14.0977 4.31802 13.5767Z"
            fill="currentColor"
          />
        </svg>
        Hold to Delete
      </div>
      <svg height="16" strokeLinejoin="round" viewBox="0 0 16 16" width="16">
        <path
          fillRule="evenodd"
          clipRule="evenodd"
          d="M6.75 2.75C6.75 2.05964 7.30964 1.5 8 1.5C8.69036 1.5 9.25 2.05964 9.25 2.75V3H6.75V2.75ZM5.25 3V2.75C5.25 1.23122 6.48122 0 8 0C9.51878 0 10.75 1.23122 10.75 2.75V3H12.9201H14.25H15V4.5H14.25H13.8846L13.1776 13.6917C13.0774 14.9942 11.9913 16 10.6849 16H5.31508C4.00874 16 2.92263 14.9942 2.82244 13.6917L2.11538 4.5H1.75H1V3H1.75H3.07988H5.25ZM4.31802 13.5767L3.61982 4.5H12.3802L11.682 13.5767C11.6419 14.0977 11.2075 14.5 10.6849 14.5H5.31508C4.79254 14.5 4.3581 14.0977 4.31802 13.5767Z"
          fill="currentColor"
        />
      </svg>
      Hold to Delete
    </button>
  );
}
```