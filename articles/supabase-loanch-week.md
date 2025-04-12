---
title: "SupabaseのLaunch Weekがあったんじゃ！"
emoji: "💰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["React","Next.js","TypeScript","Supabase",]
published: false
# published_at: 2025-04-07 09:30
---

# 初めに
私が勤めている会社ではSupabaseを積極的に採用しているので、今回はSupabaseのLaunch Weekの内容を見ていこうかなと思いまとめてみました〜
:::message
今回の記事は何かを作ったわけでもないので、さらっと「こんなことができるようになったんだな〜」という参考程度にご覧になってください！
:::

# 1日目：[Supabase UI ライブラリをリリース！](https://supabase.com/blog/supabase-ui-library)
より迅速に作業を進め、より高品質のアプリを構築するための、事前に構築されたドロップインコンポーネントのセットをリリースしたようです。
柔軟性を重視して設計されたこれらのコンポーネントは、Next.js、React Router、TanStack Start、または単純なReactアプリに自由に組み込むことができます。
またUレジストリは、いくつかの一般的なReactフレームワークで使用するために設計された再利用可能なコンポーネントのコレクションです。そのためコンポーネントはshadcn/uiとTailwindCSSでスタイル設定されており、完全にカスタマイズ可能です。
![](/images/supabase-loanch-week/image1.png)
例えばこのようなコンポーネントが含まれています。
### [パスワードベースの認証](https://arc.net/l/quote/vzizngck)
プロジェクトで認証を設定するのは複雑で時間がかかります。パスワードベースの認証ブロックは、プロジェクトで安全でユーザーフレンドリーな認証フローを数秒で実装するために必要なすべてのコンポーネントとページを提供します。
![](/images/supabase-loanch-week/image2.png)
サインアップ、サインイン、パスワードのリセット、パスワードを忘れた場合の処理​​のための完全に構築されたコンポーネントなど、開始するために必要なものがすべて含まれています。これらのコンポーネントはスタイル設定され、レスポンシブで、すぐに実稼働可能な状態になっているため、デザインやフローについて心配する必要はありません。プロジェクトにドロップするだけで、ユーザーをサインアップする準備が整います。
![](/images/supabase-loanch-week/image3.png)
### [簡単なファイルアップロード](https://arc.net/l/quote/lxbfkmuk)
ファイル アップロード ドロップゾーンコンポーネントを使用すると、数秒でアプリケーションにファイルのアップロードとストレージを追加できます。ドラッグ アンド ドロップのサポート、複数のファイルのアップロード、ファイルサイズと数の制限、画像のプレビュー、MIMEタイプの制限などの機能を備えています。
![](/images/supabase-loanch-week/image4.png)
### [リアルタイムカーソル共有](https://supabase.com/ui/docs/nextjs/realtime-cursor)
Realtime Cursorコンポーネントを使えば、アプリケーションでマルチプレイヤー体験を構築できます。このコンポーネントをプロジェクトに組み込むだけで、数秒でRealtimeを利用できるようになります。
![](/images/supabase-loanch-week/image5.png)
### [オンラインのユーザーを確認する](https://supabase.com/ui/docs/nextjs/realtime-avatar-stack)
ユーザーアバターとリアルタイムアバタースタックのコンポーネントを使えば、わずか数分でアプリにリアルタイムプレゼンスを追加できます。NotionやFigmaのように、共同作業アプリで誰がオンラインか確認できます。
![](/images/supabase-loanch-week/image6.png)
### [リアルタイムチャット](https://arc.net/l/quote/raoedgfv)
リアルタイムチャットコンポーネントは、ユーザーが共有ルーム内でリアルタイムにメッセージを交換できる、包括的なチャットインターフェースです。リアルタイムで低遅延の更新、メッセージの同期、メッセージの永続化サポート、カスタマイズ可能なメッセージ表示、新着メッセージの自動スクロール機能など、様々な機能を備えています。
![](/images/supabase-loanch-week/image7.png)

# 2日目：[ダッシュボードからのデプロイ + Deno 2.1](https://arc.net/l/quote/zpionbhz)
Supabaseダッシュボードから直接Edge Functionsを作成、テスト、編集、デプロイできるようになりました。また、本日Deno 2.1プレビュー版もリリースしました。
### Supabaseダッシュボードからエッジ関数を作成する
以前は、EdgeFunctionsを書くには、SupabaseCLIをインストールし、Dockerを起動し、エディタを Deno用に設定する必要がありました。これらの手順はもう不要です。ダッシュボードのEdgeFunctionsエディタには、DenoおよびSupabase固有のAPIの構文ハイライトと型チェック機能が組み込まれています。
![](/images/supabase-loanch-week/image8.png)
Edge Functions エディターには、Stripe WebHooks、OpenAIプロキシ、Supabaseストレージへのファイルのアップロード、電子メールの送信など、一般的なユースケースのテンプレートが含まれています。
![](/images/supabase-loanch-week/image9.png)
関数がデプロイされると、ダッシュボード内で直接編集できるようになります。また、行き詰まった場合は、インラインAIアシスタントを呼び出して説明、デバッグ、またはコードの記述を行うこともできます。
![](/images/supabase-loanch-week/image10.png)


# 3日目：

# 4日目：

# 5日目：