---
title: "もろもろインストール編"
---
さてと、まずFlutterをインストールしていきましょう。
Flutter SDKをMacにインストール（未導入なら）
```
brew install --cask flutter
```
zsh の場合、PATHを通します（まだなら
```
echo 'export PATH="$PATH:/opt/homebrew/Caskroom/flutter/latest/flutter/bin"' >> ~/.zshrc
source ~/.zshrc
```
確認
```
flutter --version
flutter doctor
```

mobileフォルダでFlutterプロジェクトを作成
```
flutter create .
```
起動確認
```
flutter pub get
flutter run
```
お疲れ様です〜
こういうのって詰まるととても時間かかりますよね〜
