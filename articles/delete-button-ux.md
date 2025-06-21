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



＃ おまけ
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