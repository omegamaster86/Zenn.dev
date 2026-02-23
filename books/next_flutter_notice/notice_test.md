---
title: "通知のテストをしてみよう"
---
またしても色々な設定が必要です。
面倒だとは思いつつも頑張りましょう〜

1. Apple 側で APNs Auth Key を作成
- Apple Developer → Certificates, Identifiers & Profiles → Keys → +
- Apple Push Notifications service (APNs) を有効化して .p8 をダウンロード
- Key ID と Team ID を控える（この3つが必要）


2. Firebase に APNs Key を登録
- Firebase Console → プロジェクト flutter-next-mobile → Project settings → Cloud Messaging
- iOS app の APNs セクションで .p8 / Key ID / Team ID を登録
- これで FCM→APNs の配送経路が有効化される