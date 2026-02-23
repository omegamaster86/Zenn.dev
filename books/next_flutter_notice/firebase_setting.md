---
title: "Firebaseの準備編"
---

さて次に通知用のFirebaseの設定を行なって行きましょう〜
1. [Firebase Console](https://console.firebase.google.com/) を開く
2. プロジェクトを追加
3. プロジェクト名を入力（例: flutter-next-mobile）
4. Google Analytics は任意（最初はOFFでもOK）
5. 作成完了後、プロジェクトを開く

このように作成できれば成功です。
![](/images/next_flutter_notice/2.png)

サイドバーの歯車アイコンをクリックし、全般をクリックしてください。
下にスクロールすると下記のような画像になるので、iosをクリックしてください。
![](/images/next_flutter_notice/9.png)
クリックしたら下記の画面に遷移するので、Xcodeで入力したbundle IDをAplleバンドルIDに入力します。
![](/images/next_flutter_notice/10.png)
「ダウンロードした GoogleService-Info.plist ファイルを Xcode プロジェクトのルートに移動し〜」とありますが、私はiosディレクトリの配下に配置しています。それでも問題はないので安心してください。
![](/images/next_flutter_notice/11.png)
さぁ次の設定をおこないましょう〜
いうて書いてある通りなので、ここでは下記の画像が表示されていれば問題なしとします。
![](/images/next_flutter_notice/12.png)
![](/images/next_flutter_notice/13.png)
![](/images/next_flutter_notice/14.png)
4つめが「初期化コードの追加」となっていますが、今回はFlutterを使用しているので、ここは何もせずスキップで良いです。
そのまま作業を進めていって、コンソールに戻った時に下記の画像と同じようになっていれば問題ないです。
![](/images/next_flutter_notice/15.png)

firebase cliがインストールされていなければ、下記を実行してください。
```
npm install -g firebase-tools
```
その後mobileディレクトリで下記のコマンドを実行してください。
```
flutter pub add firebase_core firebase_messaging
dart pub global activate flutterfire_cli
flutterfire configure
```
