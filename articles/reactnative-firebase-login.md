---
title: "ReactNativeとFirebaseを使ってログイン機能を実装"
emoji: "❄️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ReactNative","expo","Firebase"]
published: false
---

# 初めに
何故か鼻水が止まらないオメガマスターです…
今回はReactNativeとFirebaseを使ってログイン機能を実装してみました。
あまりFirebaseを使用する機会もなかったので、これを機に使用してみたいと思います。

# インストール
firebaseのインストール
`npm install firebase`
react-native-async-storageもインストール
`npm install @react-native-async-storage/async-storage`
react-native-async-storage は、React Native 向けの非同期ストレージライブラリ で、アプリ内でデータを永続化（保存）する ために使用されます。
### 主な使用用途
- ユーザーのログインセッションを保存
- 設定情報やアプリの状態をキャッシュ
- 軽量なデータ（キーと値のペア）をデバイスに保存
- ReduxやContextAPIの状態を永続化
### どんな時に使うべき？
**アプリを閉じても保持したいデータを保存する**
- 設定やユーザーデータをキャッシュ
- ローカルストレージにデータを保存し、APIリクエストを減らす
**使わない方がいい場合**
- 大きなデータを保存する（AsyncStorageはKey-Value形式なので、SQLiteやRealmの方が適している）
- セキュアなデータ（パスワードなどはSecureStoreを使う）