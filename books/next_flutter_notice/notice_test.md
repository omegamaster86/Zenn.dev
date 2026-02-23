---
title: "通知のテストをしてみよう"
---
またしても色々な設定が必要です。
面倒だとは思いつつも頑張りましょう〜

1. Apple 側で APNs Auth Key を作成
- [Apple Developer](https://developer.apple.com/account/resources/identifiers/list) → Certificates, Identifiers & Profiles → Keys → +
- Apple Push Notifications service (APNs) を有効化して .p8 をダウンロード
- Key ID と Team ID を控える（この3つが必要）
![](/images/next_flutter_notice/16.png)
![](/images/next_flutter_notice/17.png)
と進んでいくと、Download Your Keyという見出しが出てくるので、右側にあるダウンロードからダウンロードしてください。

2. Firebase に APNs Key を登録
- Firebase Console → プロジェクト flutter-next-mobile → Project settings → Cloud Messaging
- iOS app の APNs セクションで .p8 / Key ID / Team ID を登録
![](/images/next_flutter_notice/18.png)
アップロード押下後の画面
![](/images/next_flutter_notice/19.png)
これで FCM→APNs の配送経路が有効化されます。

3. iOS アプリ設定を揃える（Xcode）
- mobile/ios/Runner.xcworkspace を開く
- Runner ターゲットの Bundle ID が Firebase 登録済み iOS App と一致しているか確認
Signing & Capabilities で Push Notifications と Background Modes > Remote notifications を追加。画像の左側にある「＋ Capability」から追加できるので、「Push Notifications」と「Background Modes」をそれぞれ入力する
![](/images/next_flutter_notice/21.png)
下記のようになればOK
![](/images/next_flutter_notice/20.png)
- GoogleService-Info.plist が Runner ターゲットに含まれていることを確認
![](/images/next_flutter_notice/22.png)

4. 実機で flutter run し、権限許可＋トークン取得
- iPhone を接続して `flutter devices` IDがわかれば、`flutter run -d device_id`
- 初回の通知許可ダイアログで「許可」
- アプリで FirebaseMessaging.instance.getToken() の値をログ出力して取得
```main.dart
import 'package:firebase_messaging/firebase_messaging.dart';

  @override
  void initState() {
    super.initState();
    _logFcmToken();
  }

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  @override
  void initState() {
    super.initState();
    _logFcmToken();
  }

  Future<void> _logFcmToken() async {
    try {
      await FirebaseMessaging.instance.requestPermission();
      final String? token = await FirebaseMessaging.instance.getToken();
      debugPrint('FCM Token: $token');
    } catch (e) {
      debugPrint('Failed to get FCM token: $e');
    }
  }

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }
```
ターミナルにFCM_Tokenが表示されるので、必ずコピーすること。

5. Firebase Console からテスト送信
- Firebase Console → サイドバーのMessaging → 最初のキャンペーンを作成 → Firebase Notification メッセージを選択 → テストメッセージを送信
- 4で取得した FCM トークンを貼り付けて送信
- 受信確認（アプリは閉じておいてください。その方が通知されます。）
![](/images/next_flutter_notice/23.jpg)
ようやく通知機能が実装できましたね〜
