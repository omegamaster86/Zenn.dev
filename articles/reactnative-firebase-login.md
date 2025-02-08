---
title: "ReactNativeとFirebaseを使ってログイン機能を実装"
emoji: "❄️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ReactNative","expo","Firebase"]
published: false
---

# 初めに
何故か鼻水が止まらないオメガマスターです…
今回はReactNativeとSupabaseを使ってログイン機能を実装してみました。
今回作ってみるのはToDoアプリを実装するのでFirebaseで十分ですが、今回は実際の開発にも使用しているSupabaseを使ってみました。

# インストール
ReactNativeをインストールします。
`npx create-expo-app アプリ名`
続いてSupabaseクライアントライブラリをインストールします。
併せて必要な依存関係をインストールします。
`npx expo install @supabase/supabase-js @react-native-async-storage/async-storage @rneui/themed react-native-url-polyfill`


# いざ実装
[React NativeでSupabase Authを使用する](https://supabase.com/docs/guides/auth/quickstarts/react-native)を参考にしてみましょう。

:::message
`supabaseUrl`と`supabaseAnonKey`はそれぞれProjectSettingsのDataAPIに記載してあるので、忘れてしまった方は参照してください。
:::