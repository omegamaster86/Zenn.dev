---
title: "cursorのV1.0が出たんじゃ"
emoji: "📖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cursor","AI","AI駆動開発"]
published: false
# published_at: 
---

# 初めに
皆さん待望のcursorの正式リリース版であるV 1.0がリリースされました！
先日行われた Cursor Meetup Tokyoでは5000人以上の参加者で大賑わいでしたね（私は現地に行けなかったです）
そんな大人気のcursorで新しいバージョンが出たので内容を見ていきたいと思います。

# みなさんご存知のcursorとは？
そもそもcursorとはVS Codeをフォークした、強力なAIモデル（Claude、Gemini GPT）をエディタ内に深く統合したAIエディタです。
AIエディタが無い時代には、GPT等にコードとエラーを貼ってエラーを聞いたりしていたかもしれません。しかし、cursorはエディタ内にAIがあるので、エラーとファイルを指定して、ピンポインかつ、修正したい内容に関連したファイルも指定できるので、より効率的に問題解決や開発ができるツールです。プライバシーモードにも対応しています。
こんな感じで指定できます。
![](/images/cursor-update-v-1/image1.png)
# Cursor v1の新機能 BugBotによる自動コードレビュー
BugBotは内部的に、コミット間の差分を確認し、Cursorの最も強力なモデルを用いてコードを分析します。潜在的な問題が見つかった場合は、詳細な説明と修正の提案を含むコメントを残します。
![](/images/cursor-update-v-1/image6.png)
:::message
セットアップ後7日間の無料トライアルが付属しています。無料期間が終わった後はMaxモードと同じ価格設定になります。
支出限度額の設定もあるので、試してみてください。
[詳しくはこちら](https://docs.cursor.com/bugbot#pricing)
:::
### 設定方法
cursor.com/dashboardに遷移してサイドバーにあるIntegrationsをクリック。
Connect to GitHubをクリックしAuthorize Cursor.comをクリック
GitHubを接続していない方はManage Connectionsから連携してください。
連携後はこの画面が表示されるので、お好みで設定してください。個人的にはOnly Run when Mentionedだけで良いかなと思います。
![](/images/cursor-update-v-1/image7.png)
- Only Run when Mentioned
bugbot runPRにコメントしてBugBotを手動で起動します。
- Only Run Once
新しいコミットが追加されても、PRごとにBugBotを1回だけ実行します。
- Hide “No Bugs Found” Comments
BugBotが問題を見つけなかった場合はコメントを投稿しない

# Cursor v1の新機能 MCPワンクリックインストール

# Cursor v1の新機能 誰でも使えるバックグラウンドエージェント(Beta)
バックグラウンドエージェントを使用すると、リモート環境でコードを編集・実行できる非同期エージェントを起動できます。いつでもエージェントのステータスを確認したり、フォローアップを送信したり、引き継いだりできます。
:::message
2025/6/7時点でバックグラウンドエージェントを使用するにはプライバシーモードをオフにする必要があります。公式にはプライバシーポリシーモードをオンにしているユーザーも使用できるようにすると記載があるので、もう少し待ってみましょう。
:::


# Cursor v1の新機能 Jupyter Notebook のエージェント

# Cursor v1の新機能 Memories(Beta)

# Cursor v1の新機能 より充実したチャット応答

# Cursor v1の新機能 新しい設定とダッシュボード

# Cursor v1の新機能 より充実した差分表示

# いつからあったかわからないけどプレビュー機能
画像の真ん中にある「P」が表示されているところをクリックすると
![](/images/cursor-update-v-1/image2.png)
下記のように表示されます
![](/images/cursor-update-v-1/image3.png)
ただ今のところ完全再現はできていないようです
cursorのプレビューで表示では日本語の位置と画像の位置がおかしいですね
![](/images/cursor-update-v-1/image4.png)
zennで表示だと完璧ですね
![](/images/cursor-update-v-1/image5.png)