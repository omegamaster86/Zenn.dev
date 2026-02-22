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
さてと、まずFlutterをインストールしていきましょう。今回はFlutter側のソースは触らない予定なので、インストールしてそのままでOKです。
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

# Next.jsインストール
 Node.jsを入れる（LTS推奨）
 ```
brew install node
 ```
確認
```
node -v
npm -v
```
Next.jsプロジェクトを作成
```
npx create-next-app@latest my-next-app
```
開発サーバー起動
```
cd my-next-app
npm run dev
```