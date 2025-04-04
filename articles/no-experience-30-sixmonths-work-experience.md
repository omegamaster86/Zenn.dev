---
title: "30代実務未経験が受託開発会社に入社して半年ちょっと立った頃"
emoji: "✏️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["未経験","受託開発"]
published: true
published_at: 2025-03-16 11:00
---

# 初めに
オフィスにいると鼻水が止まらず、外に出ると鼻水が止まるオメガマスターです。
花粉症の気持ちを少しわかった気がします…
さて、今回は30代未経験者が受託開発会社に入社して気づいたことを記載しようと思います。
未経験の方は私のようにならないように、経験者は新人が私のようになっていないかチェックしてあげてください。

# 質問をめっちゃしよう
実務未経験者が作成するコードと経験者が作成するコードには当然違いがありますが、それ以前に前提知識や着眼点が異なっていることが多いです。
方向性の確認の意味でも確認してその後、AIを使用しましょう。

近年では生成AIやAI Editorで簡単にコードの記載や質問が出来ます。
そのため上司に質問しなくてもAIに聞けばある程度の正解が返ってきます。もちろん使い手の指示やプロンプトによっても結果が左右されるので、皆さんが同じ解答を得られるとは限りません…
例えばこんな質問をGTPに聞くとします。
前提条件
バックエンドやSQLをあまり理解しておらず、フロントエンド、バックエンドの責務をあまりわかっていない社員で、SQLにはこんなコードが記載してある`SELECT * FROM t_users`

```
バックエンドから取得しているデータをフロントエンドで表示したい
複数データがあるのである分だけ表示したいので、mapを使ってほしい
既に退会している人は表示しないで
表示するものは〜〜
```
とします。もちろんGTPは指示内容に従ってコードを生成してくれますが…
おそらく完成したコードで動いたとしても、上司からグーパンが飛んでくるでしょう…
今回はデータを取得するSQLを修正するべきです。もちろん結果的に画面に表示されるものに違いはありませんが、品質や責務といった点でだいぶ異なる結果になります。
GTPに正しく情報を与えていれば「修正するべきはフロントではなくSQLです。修正案は〜」となりますが、使用者が正しく指示出していなければ必ずしも良い結果になるとは限りません。
そしてこういうのは上司に質問すればすぐに指摘してもらえます。「こういうことをしたいのですが、フロントを修正する方向で良いですか？」と質問したらほぼ全ての上司が「いやSQLを修正しよう」と言うでしょう。

# ベンチャー企業 or 社員があまりいない会社向け
フロントエンドだけ、バックエンドだけより、いろいろ出来た方が市場価値も上がるので、フロントバックどっちもやれる会社に入社出来たら、むしろラッキーと思いましょう！！

私はフロントエンドはフロントエンドだけ、バックエンドはバックエンドだけ、インフラはインフラだけを作業し、各PJにそれぞれの担当がいると思っていました。
もちろん大きい会社なら上記のような感じかもしれません（ここ適当なので、間に受けないように！！）
しかし私が入社した会社ではフロント、バックどっちもやるというパターンでした。（正直かなりびっくりしました）
しかし市場価値が上がるかもと思い、むしろラッキーぐらいな感覚で今は楽しんでいます。それにSQLを学ぶとデータ取得がだいぶ楽だなと実感もあるので、これからIT業界にチャレンジする方はSQLの知識は必須かもしれません。ぜひ学んでおきましょう。

# 案件が終わったら、案件で作成したアプリのミニ版を作成してみよう！
これは完全に趣味の領域にもなってしまいますが、チーム開発だと自分がまだ開発したことない機能を他の方が開発している場合もあると思います。
そう言った時にミニアプリを作成して、自分が実装したことない機能を実装してみましょう！
ライブラリを使用した場合、自作でコンポーネントを作成したら、別の案件で使用も出来ますのでやって損はないかと思います。
これ意外と転職活動で評価されると思いますよ！

# AIツールを使用しよう
さまざまなAIツールが出てきている昨今では、むしろ使ったことがない人の方が少ないのではないでしょうか？今後の面接ではAIを使用した経験だったり、こういう業務をAIを使用して効率化した。みたいな経験も必要になってくると思います。
wantedly等で会社情報の中にAIを使用している可能性があるなら「御社ではどのように使用しているか」と逆質問もしやすいです。

# 最後に
この記事が未経験者の　役に立てれば幸いです。