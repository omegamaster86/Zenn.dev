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

