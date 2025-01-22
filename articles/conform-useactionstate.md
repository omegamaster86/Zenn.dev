---
title: "Server Actionsに使用するConform（ライブラリ）とuseActionState"
emoji: "🗒️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "ecmascript"]
published: false
---

# 初めに
さて皆さん、インフルエンザが猛威を振るっておりますが、体調はいかがですか？
私は年末にA型インフルエンザにかかり、しんどい思いをしていました。
体調管理には気をつけてください！

# 今回書く内容と現状のPJ
皆さんはNext.jsのServer Actionsを使用していますか？
私は最近App RouterのPJ（去年の開発の続きのPJ）に参画しているのですが、
useSWRでデータ取得/更新をしたり、useEffectでデータ取得/更新しているPJです…
useEffectでデータ取得/更新は皆さんしていないと思いますが、useSWRでデータ取得/更新と言われて「おいおい！」
と言える方は何人いらっしゃいますか？
正直私も「そういや昔使っていたな〜」と思いながら開発をしていました。
しかし、App Routerを使用しているのにServer Actionsをしていないとは何事だ！
と思ったので、今回はServer Actionsを使用してデータ取得/更新をする方法を紹介します。

# Server Actionsとは