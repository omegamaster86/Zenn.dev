---
title: "新エディタのkiroとcursorを比較してみたんじゃ(コーディング編)"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["kiro","cursor","AI","AI駆動開発"]
published: false
published_at: 2025-07-19 19:30
---
# はじめに
この記事は下記の記事の続きとなっておりますので、もしよければこちらもご覧ください。
https://zenn.dev/omegamaster/articles/kiro-cursor-diference
分けなくてもよかったかもと書き終えてから思いました…

# 結論
- Agent Hooksは素晴らしい！
- 実装スピードが遅いとXで言われていますが、そんな気にならないかも
- AI駆動開発はコンテキストや仕様が命
- やっぱりタブ補完、コミットメッセージの自動生成は欲しい
- 正式にリリースした際の費用感次第で使用の有無が分かれる気がする

# コードを変更した際のAgent Hooksの挙動
今回は試しにNext.jsをセットアップした際のトップ画面を編集してみました。（適当にタグ追加しただけなので、コードは割愛）すると下記の動作をAgentが行ってくれました。
cursorのrulesだと実行してくれないこともあるので、Agent Hooksはかなりポイント高いですね！
※前回の記事でファイルを変更したら、テストとドキュメント更新する指示を与えています。
![](/images/kiro-cursor-diference-two/image1.png)
![](/images/kiro-cursor-diference-two/image2.png)
![](/images/kiro-cursor-diference-two/image3.png)
:::details 作成されたREADME
# kiro-test

Next.js 15とReact 19を使用したモダンなWebアプリケーションプロジェクトです。

## 技術スタック

- **Next.js 15.4.1** - App Routerを使用したReactフレームワーク
- **React 19.1.0** - 最新のReactとconcurrent機能
- **TypeScript 5** - 型安全なJavaScript
- **Tailwind CSS 4** - ユーティリティファーストCSSフレームワーク
- **Turbopack** - 高速な開発用バンドラー

## 開発環境のセットアップ

### 前提条件
- Node.js 18以上
- npm または yarn

### インストール

```bash
npm install
```

### 開発サーバーの起動

```bash
npm run dev
```

ブラウザで [http://localhost:3000](http://localhost:3000) を開いて結果を確認してください。

### その他のコマンド

```bash
# プロダクションビルド
npm run build

# プロダクションサーバー起動
npm run start

# コードリンティング
npm run lint
```

## プロジェクト構造

```
├── app/                 # Next.js App Router
│   ├── layout.tsx      # ルートレイアウト
│   ├── page.tsx        # ホームページ
│   └── globals.css     # グローバルスタイル
├── public/             # 静的アセット
└── .kiro/              # Kiro設定ファイル
```

## 開発ガイドライン

- **ファイル命名**: ケバブケースを使用（例: `button.tsx`）
- **コンポーネント名**: パスカルケースを使用（例: `HeaderBreadcrumb`）
- **スタイリング**: Tailwind CSSユーティリティクラスを優先
- **TypeScript**: 厳密モードを有効化
- **レスポンシブデザイン**: モバイルファーストアプローチ

## デプロイ

このプロジェクトは[Vercel](https://vercel.com/)での簡単なデプロイに最適化されています。

詳細については[Next.js deployment documentation](https://nextjs.org/docs/app/building-your-application/deploying)を参照してください。
:::
やっぱりこのドキュメント生成力は本当に強い！

# パッケージインストール(今回はJest)に失敗した際の挙動
AIにパッケージインストールしてもらうことは多々あると思います。kiroでは`Test Suites: 1 failed,`と出力されると依存関係を調べ、`Pass`するまで実行してくれますが、cursorだと最近`fail`でも完了しました。エラーが起きてても「実装が完了しました」と出力される時があるので、正確性はkiroの方がcursorより良い印象です。
そして最後にエラーの原因をまとめてくれます。
チャットのやり取りを遡らなくて良いので、これは便利ですね〜
昔のcursorではエラーが起きていたら「エラーが発生しているので、原因を特定し修正します」ぐらい言ってきた気がしますが…新料金発表あたりから、様子が以前と異なりますね。
![](/images/kiro-cursor-diference-two/image4.png)

# Task Listの実装
:::details 以前kiroが出力したTask List
# 実装計画

- [ ] 1. プロジェクトセットアップとデータベース設定
  - Prismaをインストールし、SQLiteデータベースを設定する
  - Todoモデルのスキーマを定義し、マイグレーションを実行する
  - Prismaクライアントの初期化とデータベース接続を確認する
  - _要件: 1.3, 2.2_

- [ ] 2. Server Actionsの実装
  - [ ] 2.1 基本的なServer Actionsファイルを作成
    - lib/actions.tsファイルを作成し、Prismaクライアントをインポートする
    - 'use server'ディレクティブを追加し、基本構造を設定する
    - _要件: 1.1, 1.3_

  - [ ] 2.2 createTodo Server Actionを実装
    - FormDataからタイトルを取得し、バリデーションを行う
    - 新しいタスクをデータベースに保存する機能を実装する
    - revalidatePathでページを更新する処理を追加する
    - _要件: 1.1, 1.2_

  - [ ] 2.3 toggleTodo Server Actionを実装
    - タスクIDを受け取り、完了状態を切り替える機能を実装する
    - データベースの更新とrevalidatePathを実行する
    - _要件: 3.1, 3.2_

  - [ ] 2.4 updateTodo Server Actionを実装
    - タスクIDとタイトルを受け取り、タスクを更新する機能を実装する
    - バリデーションとデータベース更新処理を実装する
    - _要件: 5.1, 5.2, 5.4_

  - [ ] 2.5 deleteTodo Server Actionを実装
    - タスクIDを受け取り、タスクを削除する機能を実装する
    - データベースからの削除とrevalidatePathを実行する
    - _要件: 4.2_

- [ ] 3. データ取得関数の実装
  - [ ] 3.1 getTodos関数を実装
    - Prismaクライアントを使用して全タスクを取得する関数を作成する
    - 作成日時の降順でソートする処理を追加する
    - _要件: 2.1, 2.2_

  - [ ] 3.2 フィルタリング機能付きgetTodos関数を実装
    - フィルタータイプ（all, active, completed）を受け取る機能を追加する
    - 条件に応じてタスクをフィルタリングする処理を実装する
    - _要件: 6.1, 6.2, 6.3_

- [ ] 4. UIコンポーネントの実装
  - [ ] 4.1 TodoFormコンポーネントを作成
    - フォーム要素とcreateToDoアクションを統合するClient Componentを作成する
    - バリデーション機能とエラー表示を実装する
    - 送信後のフォームリセット機能を追加する
    - _要件: 1.1, 1.2_

  - [ ] 4.2 TodoItemコンポーネントを作成
    - 個別タスクの表示とアクション（完了切り替え、削除）を実装する
    - インライン編集機能を持つClient Componentを作成する
    - 完了済みタスクの取り消し線スタイルを実装する
    - _要件: 3.1, 3.2, 3.3, 4.1, 4.2, 4.3, 5.1, 5.2, 5.3_

  - [ ] 4.3 TodoListコンポーネントを作成
    - タスクリストを表示するServer Componentを作成する
    - 各TodoItemコンポーネントをレンダリングする処理を実装する
    - タスクが存在しない場合のメッセージ表示を追加する
    - _要件: 2.1, 2.2, 2.3_

  - [ ] 4.4 TodoFilterコンポーネントを作成
    - フィルタリングオプション（全て/未完了/完了済み）のClient Componentを作成する
    - URLパラメータでフィルター状態を管理する機能を実装する
    - アクティブフィルターの視覚的表示を追加する
    - _要件: 6.1, 6.2, 6.3_

- [ ] 5. メインページの統合
  - [ ] 5.1 TodoPageコンポーネントを作成
    - Server Componentとしてメインページを実装する
    - URLパラメータからフィルター状態を取得する処理を追加する
    - getTodos関数を呼び出してデータを取得する
    - _要件: 2.1, 6.1, 6.2, 6.3_

  - [ ] 5.2 全コンポーネントを統合
    - TodoPage内でTodoForm、TodoFilter、TodoListコンポーネントを統合する
    - 適切なpropsの受け渡しとレイアウトを実装する
    - _要件: 全要件の統合_

- [ ] 6. スタイリングとUX改善
  - [ ] 6.1 Tailwind CSSでレスポンシブデザインを実装
    - モバイルファーストのレスポンシブレイアウトを作成する
    - 美しいUIデザインとアニメーション効果を追加する
    - _要件: 全要件のUX向上_

  - [ ] 6.2 エラーハンドリングとローディング状態を実装
    - フォーム送信時のローディング状態を表示する機能を追加する
    - エラーメッセージの適切な表示を実装する
    - _要件: 1.2, 5.4_

- [ ] 7. テストの実装
  - [ ] 7.1 Server Actionsの単体テストを作成
    - createTodo、updateTodo、deleteTodo、toggleTodoのテストを実装する
    - バリデーション機能のテストを追加する
    - _要件: 全要件の品質保証_

  - [ ] 7.2 コンポーネントのテストを作成
    - 各UIコンポーネントのレンダリングテストを実装する
    - ユーザーインタラクションのテストを追加する
    - _要件: 全要件の品質保証_
:::
Task ListでStart taskをクリックすると2枚目のようにsettingsの中身を自動的に見にいってくれます。当然足りないパッケージ等はインストールしてくれます。
![](/images/kiro-cursor-diference-two/image6.png)
![](/images/kiro-cursor-diference-two/image5.png)
簡易的なテストファイルも作成し、動作確認して問題がなければ削除もしてくれます。
![](/images/kiro-cursor-diference-two/image7.png)
Taskが完了すると
![](/images/kiro-cursor-diference-two/image8.png)
あとはTask Listが全てcompletedになるまで繰り返していきます！
もちろん実装後に毎回Jestでテストもしてくれています！
Xには実装時間が遅いと書かれていますが、そこまで気になりませんでした…皆さんAIがコード生成している時はX見たり、FA⚫︎ZA見たりしているので、そんなに気にしなくても良いかと…

# 完成
![](/images/kiro-cursor-diference-two/image9.png)
Jestでもテストしていたおかげでエラーなく、仕様どうりの挙動になっています。
またAI駆動開発はコンテキストが命なので、仕様書をしっかり定義するとしっかり実装してくれます。
しかし、steeringフォルダの内容が常に反映されているかと言われると怪しいので、ここは他のツール同様適宜確認してあげてください。

# 最後に
この記事が誰かの役に立てれば幸いです。