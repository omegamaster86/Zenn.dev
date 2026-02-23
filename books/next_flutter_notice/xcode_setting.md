---
title: "xcode準備編"
---
さて、.cursorと.gitignore以外は、おそらく皆さんのディレクトリはこんな感じになっていると思います。
![](/images/next_flutter_notice/3.png)
Firebaseの設定でbundle ID というものが必要になりますので、先にXcodeでbundle ID を設定してしまいましょう〜
App Storeで「Xcode」と検索して、下記の画像のアプリをインストールしてください。
![](/images/next_flutter_notice/4.png)
TestFlightというテスト配信アプリも後に使用するので、今のうちにインストールしてください。
![](/images/next_flutter_notice/5.png)
Xcodeを開くとこんな感じの画面になると思います。
![](/images/next_flutter_notice/6.png)
Open Existing Projectで作成した フォルダのiosディレクトリまで移動してから、開いてください。
![](/images/next_flutter_notice/7.png)
この画面にあるBundle Identifierと後に設定するFirebaseのbundle ID が一致している必要があるので、注意してください。
![](/images/next_flutter_notice/8.png)
ここまでくれば一旦Xcodeの設定はOKなので、Firebaseの設定に移りましょう~
