---
title: "Next.16で改善されたキャッシュAPIを調査したんじゃ"
emoji: "✏️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Next.js","cash"]
published: false
# published_at: 2025-11-01 21:30
publication_name: "genai"
---
# 初めに
気づいたら11月、今年ももう少しで終わりですね〜
どうでもいいんですけど、Fischer’sの増量中を会社でやりたいなと密かに思っています。（絶対にここで書く事ではないのだけど）
さて今回はNext.16で改善されたキャッシュAPI主にrevalidateTag・updateTag・refreshの使い分けや使い方を詳しく見ていこうと思います。

