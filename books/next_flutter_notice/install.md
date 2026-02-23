---
title: "もろもろインストール編"
---
この本を見る人は既にFlutterやNextが入っていそうですが、一応記載します。

# パッケージ管理ツールをインストール
Homebrewを入れる（未導入の場合）
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
確認
```
brew --version
```

# Flutterインストール
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
まぁCursorやClaude Codeに依頼すれば対応してくれるので、やってもらった方が楽だと思います。
さぁ次に行きましょう〜