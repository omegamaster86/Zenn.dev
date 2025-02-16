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
さて今回は巷で話題のHonoを使ってみました。
使ってみたと言っても、Honoのデータを表示させるだけなので、初学者向けにはなりますが、
とっかかりとして見ていって頂けたら幸いです。

# フォルダ構成
```
.
├── apps
│   ├── backend
│   │   ├── package.json
│   │   ├── src
│   │   │   ├── app.ts
│   │   │   ├── index.local.ts
│   │   │   ├── index.ts
│   │   │   ├── infrastructure
│   │   │   │   └── prisma
│   │   │   │       └── employees
│   │   │   └── presentation
│   │   │       └── routes
│   │   │           └── employees
│   │   └── tsconfig.json
│   │   └── vitest.config.ts
│   └── frontend
│       ├── index.html
│       ├── package.json
│       ├── src
│       │   ├── main.tsx
│       ├── tsconfig.json
│       └── vite.config.ts
```