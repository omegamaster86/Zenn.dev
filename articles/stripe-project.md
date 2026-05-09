---
title: "Stripe Projectを試してみたんじゃ"
emoji: "🌕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Stripe Project","AI","AI駆動開発"]
published: false
# published_at: 2026-05-08 09:30
publication_name: "genai"
---
# 初めに


# 使用してみよう
### インストール
```
brew install stripe/stripe-cli/stripe && stripe plugin install projects
```
公式にあったskillsをインストール
```
npx skills add https://docs.stripe.com
```
実行するとこんな感じの選択画面が表示されます。
![](/images/stripe-project/1.png)

| スキル | 意味 | 選ぶか |
| --- | --- | --- |
| stripe-projects | Projects / CLI まわりのガイド | 選ぶ（これがメイン） |
| stripe-best-practices | Stripe 連携の一般的なベストプラクティス | 任意（Stripe の実装もするならおすすめ） |
| upgrade-stripe | API バージョンや SDK のアップグレード手順 | 今回の「Projects を試す」だけなら不要。バージョン上げを検討するときに入れればよい |

Stripeの機能は必要としないので、stripe-projectsを選択していきます。 
選択するにはスペースをクリックする必要があるようですね。
![](/images/stripe-project/2.png)
選択して、Enterをクリックするとこんな選択肢が表示されます。
![](/images/stripe-project/3.png)
32もエージェントがあるんだと思う一方でcursorがなかったので、納得がいきません。激おこです。
何も選ばずにEnterをクリックすると、どこまで適応させるか選択することができます。
今回はProjectを選択しておきましょう。
![](/images/stripe-project/4.png)

