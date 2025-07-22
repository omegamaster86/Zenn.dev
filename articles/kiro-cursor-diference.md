---
title: "新エディタのkiroとcursorを比較してみたんじゃ"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["kiro","cursor","AI","AI駆動開発"]
published: true
published_at: 2025-07-17 17:30
---

# はじめに
気が早いかもしれませんが、そろそろ梅雨も終わり、夏も本番を迎えてきましたね〜
さてAWSから新しいエディタのkiroがプレビュー版でリリースされました！
早速見ていきながら、cursorと比較してみましょう〜
思ったより画像が多くなり、機能紹介編、実装編と分けさせてください🙇
:::message
この記事は2025/07/17までの情報です。
細真の情報は公式ドキュメントを必ず参照してください。
:::

# 結論
- まだプレビュー版なので、これからどんどん情報が追加されていくと思うので、要チェックや！（とあるバスケ漫画のマネージャー風）
- claude codeとkiroの併用が強いかもその場合、もしかしたらkiroが無料で使用できるかも？
-必ず実行されるAgent Hooksは.cursor/rulesより遥かに強力！

# そもそもkiroって？
 IDE内蔵型のAIコーディングエージェントです。Kiroの特徴は他のエディタにもあるVibe(バイブコーディング)だけでなく、Spec（要件定義・設計を先にやる）を使用することで要件定義・設計からコード生成を行なってくれます。
 もうやることが決まっていて、コードだけ作って欲しいならVibeモードで、これから何か作るという方はSpecで要件・設計を決めてから実装の方がより実務に近くなるので、万人におすすめです。
https://kiro.dev/index
初期セットアップは画像が多くなるので、まず何ができるのか、どんな感じなのかみてきましょう〜

# 触ってみよう
:::message
あくまでkiroはまだプレビュー版ということをお忘れなく！
これしか出来ないのか〜ではなく、プレビューでここまでできるんだ〜という観点で見ていきましょう！
:::
ターミナルで開発用フォルダに移動し、`npx create-next-app`を実行します。（ここは好きな言語やフレームワークを使用してください）
もろもろ終わった後に`cd`でフォルダに移動して`kiro .`で起動させます。下記の画像が表示されるので、信用してあげてください。信頼すると2枚目の画像の画面に遷移します。
![](/images/kiro-cursor-diference/image1.png)
![](/images/kiro-cursor-diference/image2.png)
現在はモデルがsonnet3.7とsonnet4しか選択できませんが、モデルは今後追加されていくと思うので待ちましょう。大丈夫、彼女待つ時間よりは短いと思いますよ…
AutoPilotをオンにすると承認なしでどんどん実装をしていきますが、フォルダを消される恐れがあるので、一旦オフにすることをおすすめします。
Specに下記のようにお願いしました。
```
ToDoリストを作成したい。バックエンドとDBの選定は任せます。
```
その後.kiro/specs/todo-list/requirements.mdが作成され、要件が適切か聞いてきます。もし要件に変更があれば手修正でもkiroにお願いして修正でも大丈夫です。
![](/images/kiro-cursor-diference/image3.png)
:::details kiro作成の要件定義内容
# 要件定義書

## 概要

ユーザーがタスクを管理できるToDoリストアプリケーションです。タスクの作成、編集、削除、完了状態の管理を行い、効率的なタスク管理を支援します。

## 要件

### 要件1

**ユーザーストーリー:** ユーザーとして、新しいタスクを作成したい。そうすることで、やるべきことを記録できる。

#### 受け入れ基準

1. WHEN ユーザーがタスク作成フォームにタイトルを入力し送信ボタンを押す THEN システムは新しいタスクを作成し、タスクリストに表示する
2. WHEN ユーザーが空のタイトルでタスクを作成しようとする THEN システムはエラーメッセージを表示し、タスクを作成しない
3. WHEN ユーザーがタスクを作成する THEN システムはタスクに一意のIDと作成日時を自動で割り当てる

### 要件2

**ユーザーストーリー:** ユーザーとして、作成したタスクの一覧を見たい。そうすることで、現在のタスク状況を把握できる。

#### 受け入れ基準

1. WHEN ユーザーがアプリケーションにアクセスする THEN システムは全てのタスクをリスト形式で表示する
2. WHEN タスクリストが表示される THEN システムは各タスクのタイトル、作成日時、完了状態を表示する
3. WHEN タスクが存在しない THEN システムは「タスクがありません」というメッセージを表示する

### 要件3

**ユーザーストーリー:** ユーザーとして、タスクを完了済みにマークしたい。そうすることで、完了したタスクを区別できる。

#### 受け入れ基準

1. WHEN ユーザーが未完了のタスクのチェックボックスをクリックする THEN システムはそのタスクを完了済みとしてマークする
2. WHEN ユーザーが完了済みのタスクのチェックボックスをクリックする THEN システムはそのタスクを未完了に戻す
3. WHEN タスクが完了済みになる THEN システムはタスクのタイトルに取り消し線を表示する

### 要件4

**ユーザーストーリー:** ユーザーとして、不要になったタスクを削除したい。そうすることで、タスクリストを整理できる。

#### 受け入れ基準

1. WHEN ユーザーがタスクの削除ボタンをクリックする THEN システムは確認ダイアログを表示する
2. WHEN ユーザーが削除を確認する THEN システムはタスクを完全に削除し、タスクリストから除去する
3. WHEN ユーザーが削除をキャンセルする THEN システムはタスクを削除せず、元の状態を維持する

### 要件5

**ユーザーストーリー:** ユーザーとして、タスクの内容を編集したい。そうすることで、タスクの詳細を更新できる。

#### 受け入れ基準

1. WHEN ユーザーがタスクのタイトルをクリックする THEN システムはインライン編集モードに切り替える
2. WHEN ユーザーが編集を完了しEnterキーを押すかフォーカスを外す THEN システムはタスクのタイトルを更新する
3. WHEN ユーザーがEscキーを押す THEN システムは編集をキャンセルし、元のタイトルに戻す
4. IF ユーザーが空のタイトルで更新しようとする THEN システムはエラーメッセージを表示し、更新を拒否する

### 要件6

**ユーザーストーリー:** ユーザーとして、タスクをフィルタリングしたい。そうすることで、特定の状態のタスクのみを表示できる。

#### 受け入れ基準

1. WHEN ユーザーが「全て」フィルターを選択する THEN システムは全てのタスクを表示する
2. WHEN ユーザーが「未完了」フィルターを選択する THEN システムは未完了のタスクのみを表示する
3. WHEN ユーザーが「完了済み」フィルターを選択する THEN システムは完了済みのタスクのみを表示する
:::
問題なかったので、「適切です」と送ったら下記の画像のようになりました。
どのファイルを読み込んでいるかわかるのが良いですね〜cursorには無いんじゃないかな？（あったらごめんなさい）
![](/images/kiro-cursor-diference/image4.png)
:::details kiro作成の設計書
# 設計書

## 概要

Next.js App Routerを使用したフルスタックToDoリストアプリケーション。フロントエンドはReact 19とTailwind CSS、バックエンドはNext.js API Routes、データベースはSQLiteとPrisma ORMを使用します。

## アーキテクチャ

### 技術スタック
- **フロントエンド**: React 19, Next.js 15 (App Router), TypeScript, Tailwind CSS
- **バックエンド**: Next.js API Routes
- **データベース**: SQLite
- **ORM**: Prisma
- **状態管理**: React useState/useEffect (シンプルなアプリのため)

### アーキテクチャパターン
- **フロントエンド**: コンポーネントベースアーキテクチャ
- **バックエンド**: RESTful API設計
- **データアクセス**: Repository パターン（Prismaクライアント経由）

## コンポーネントとインターフェース

### フロントエンドコンポーネント

#### 1. TodoApp (メインコンポーネント)
- 全体の状態管理
- API呼び出しの調整
- 子コンポーネントの統合

#### 2. TodoForm
- 新しいタスクの作成フォーム
- バリデーション機能
- 送信処理

#### 3. TodoList
- タスクリストの表示
- フィルタリング機能
- 各タスクアイテムの管理

#### 4. TodoItem
- 個別タスクの表示
- インライン編集機能
- 完了状態の切り替え
- 削除機能

#### 5. TodoFilter
- フィルタリングオプション（全て/未完了/完了済み）
- アクティブフィルターの表示

### API エンドポイント

#### GET /api/todos
- 全タスクの取得
- クエリパラメータでフィルタリング対応

#### POST /api/todos
- 新しいタスクの作成
- リクエストボディ: `{ title: string }`
- レスポンス: 作成されたタスクオブジェクト

#### PUT /api/todos/[id]
- タスクの更新（タイトル、完了状態）
- リクエストボディ: `{ title?: string, completed?: boolean }`
- レスポンス: 更新されたタスクオブジェクト

#### DELETE /api/todos/[id]
- タスクの削除
- レスポンス: 削除成功メッセージ

## データモデル

### Todo エンティティ

```typescript
interface Todo {
  id: string;          // UUID
  title: string;       // タスクのタイトル
  completed: boolean;  // 完了状態
  createdAt: Date;     // 作成日時
  updatedAt: Date;     // 更新日時
}
```

### Prismaスキーマ

```prisma
model Todo {
  id        String   @id @default(cuid())
  title     String
  completed Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

## エラーハンドリング

### フロントエンド
- API呼び出し失敗時のエラー表示
- フォームバリデーションエラーの表示
- ネットワークエラーの適切な処理

### バックエンド
- 400: バリデーションエラー（空のタイトルなど）
- 404: 存在しないタスクへのアクセス
- 500: サーバー内部エラー

### エラーレスポンス形式
```typescript
interface ErrorResponse {
  error: string;
  message: string;
  statusCode: number;
}
```

## テスト戦略

### 単体テスト
- コンポーネントのレンダリングテスト
- API エンドポイントの機能テスト
- バリデーション機能のテスト

### 統合テスト
- フロントエンドとバックエンドの連携テスト
- データベース操作のテスト

### E2Eテスト
- ユーザーフローの完全なテスト
- ブラウザでの実際の操作テスト

### テストツール
- **単体テスト**: Jest, React Testing Library
- **E2Eテスト**: Playwright または Cypress

## セキュリティ考慮事項

- SQLインジェクション対策（Prismaによる自動対策）
- XSS対策（Reactによる自動エスケープ）
- CSRF対策（Next.jsのデフォルト設定）
- 入力値のサニタイゼーション

## パフォーマンス最適化

- Server-side Rendering（SSR）の活用
- 静的生成（SSG）の検討
- 画像最適化（Next.js Image コンポーネント）
- バンドルサイズの最適化

## デプロイメント

- **開発環境**: `npm run dev`
- **本番環境**: Vercel または類似のプラットフォーム
- **データベース**: 本番環境ではPostgreSQLまたはMySQLへの移行を検討
:::
useEffectは基本的に使用しないで欲しい。他は問題ない。と記載したところ、再度設計書を作成してくれました。
![](/images/kiro-cursor-diference/image5.png)
:::details 指摘後の設計書
# 設計書

## 概要

Next.js App Routerを使用したフルスタックToDoリストアプリケーション。フロントエンドはReact 19とTailwind CSS、バックエンドはNext.js API Routes、データベースはSQLiteとPrisma ORMを使用します。

## アーキテクチャ

### 技術スタック
- **フロントエンド**: React 19, Next.js 15 (App Router), TypeScript, Tailwind CSS
- **バックエンド**: Next.js Server Actions
- **データベース**: SQLite
- **ORM**: Prisma
- **状態管理**: Server Components + Server Actions (Next.js 15の推奨パターン)

### アーキテクチャパターン
- **フロントエンド**: Server Components + Client Components
- **バックエンド**: Server Actions パターン
- **データアクセス**: Prismaクライアント直接アクセス

## コンポーネントとインターフェース

### フロントエンドコンポーネント

#### 1. TodoPage (Server Component)
- データの取得とレンダリング
- Server Actionsの統合
- 子コンポーネントの統合

#### 2. TodoForm (Client Component)
- 新しいタスクの作成フォーム
- Server Actionを使用した送信処理
- バリデーション機能

#### 3. TodoList (Server Component)
- タスクリストの表示
- フィルタリング機能
- 各タスクアイテムの管理

#### 4. TodoItem (Client Component)
- 個別タスクの表示
- インライン編集機能
- Server Actionsを使用した完了状態切り替え
- Server Actionsを使用した削除機能

#### 5. TodoFilter (Client Component)
- フィルタリングオプション（全て/未完了/完了済み）
- URLパラメータでの状態管理

### Server Actions

#### createTodo
- 新しいタスクの作成
- パラメータ: `FormData` (title フィールド)
- 戻り値: void (revalidatePathでページを更新)

#### updateTodo
- タスクの更新（タイトル、完了状態）
- パラメータ: `id: string, data: { title?: string, completed?: boolean }`
- 戻り値: void (revalidatePathでページを更新)

#### deleteTodo
- タスクの削除
- パラメータ: `id: string`
- 戻り値: void (revalidatePathでページを更新)

#### toggleTodo
- タスクの完了状態切り替え
- パラメータ: `id: string`
- 戻り値: void (revalidatePathでページを更新)

## データモデル

### Todo エンティティ

```typescript
interface Todo {
  id: string;          // UUID
  title: string;       // タスクのタイトル
  completed: boolean;  // 完了状態
  createdAt: Date;     // 作成日時
  updatedAt: Date;     // 更新日時
}
```

### Prismaスキーマ

```prisma
model Todo {
  id        String   @id @default(cuid())
  title     String
  completed Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

## エラーハンドリング

### フロントエンド
- API呼び出し失敗時のエラー表示
- フォームバリデーションエラーの表示
- ネットワークエラーの適切な処理

### バックエンド
- 400: バリデーションエラー（空のタイトルなど）
- 404: 存在しないタスクへのアクセス
- 500: サーバー内部エラー

### エラーレスポンス形式
```typescript
interface ErrorResponse {
  error: string;
  message: string;
  statusCode: number;
}
```

## テスト戦略

### 単体テスト
- コンポーネントのレンダリングテスト
- API エンドポイントの機能テスト
- バリデーション機能のテスト

### 統合テスト
- フロントエンドとバックエンドの連携テスト
- データベース操作のテスト

### E2Eテスト
- ユーザーフローの完全なテスト
- ブラウザでの実際の操作テスト

### テストツール
- **単体テスト**: Jest, React Testing Library
- **E2Eテスト**: Playwright または Cypress

## セキュリティ考慮事項

- SQLインジェクション対策（Prismaによる自動対策）
- XSS対策（Reactによる自動エスケープ）
- CSRF対策（Next.jsのデフォルト設定）
- 入力値のサニタイゼーション

## パフォーマンス最適化

- Server-side Rendering（SSR）の活用
- 静的生成（SSG）の検討
- 画像最適化（Next.js Image コンポーネント）
- バンドルサイズの最適化

## デプロイメント

- **開発環境**: `npm run dev`
- **本番環境**: Vercel または類似のプラットフォーム
- **データベース**: 本番環境ではPostgreSQLまたはMySQLへの移行を検討
:::
だいぶしっかり作成してくれますね〜
その後「適切です」とクリックすると実行計画書を作成してくれました。
:::details kiroが作成した実行計画書
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
今回はどのように動作するか見たいので、一旦OKとします。OKと伝えると下記の画像のように準備完了と出力されます。
![](/images/kiro-cursor-diference/image6.png)
なかなかしっかり出来上がっている気がしますね〜
会社毎の仕様書フォーマットを読み込ませて、なぞるように記載させれば、結構な手間が削減できるのではないでしょうか？
あと現在実装速度が遅いらしいので、ここまでkiroを使用して、この後はclaude codeを使用するのが良いと思います。
詳しくはこちらの記事に託します。
https://zenn.dev/ubie_dev/articles/kiro-claude-code

# 実装の前に
サイドバーのゴーストアイコンをクリックすると下記の画面が表示されます
![](/images/kiro-cursor-diference/image7.png)
### SPEC
SPECには先ほど作成した仕様書が見れるようになっています。
![](/images/kiro-cursor-diference/image8.png)
真ん中にある1,2,3のボタンをクリックするとそれぞれの定義書に遷移します。
### Agent Hooks
Agent Hooksは、IDEで特定のイベントが発生した際に、事前定義されたエージェントアクションを実行する自動トリガーです。定型的なタスクの実行を手動で指示するのではなく、フックは次のようなイベントへの自動応答を設定します。
- ファイルの保存
- 新しいファイルの作成
- ファイルの削除

エージェントフックは、インテリジェントな自動化によって開発ワークフローを変革します。一般的なタスクにフックを設定することで、以下のことが可能になります。
- 貫したコード品質を維持する
- セキュリティの脆弱性を防ぐ
- 手作業のオーバーヘッドを削減
- チームのプロセスを標準化する
- 開発サイクルを高速化
じゃ実際に中身を見ていきましょう〜
Agent Hooks横の＋ボタンをクリックすると画像のような画面が表示されます。
![](/images/kiro-cursor-diference/image9.png)
Describe a hook using natural languageに指定したいことを記載しましょう。私はこんな感じで記載しました。
```
ファイルが変更されたら必要に応じてドキュメントを更新して、単体テストを実施して欲しい。
```
その後エンターをクリックするとAgent Hooksが作成されました。
![](/images/kiro-cursor-diference/image10.png)
:::message
タスクリストがすでにある状態で、Agent Hooksを追加しようとすると順番待ちみたいになるので、タスクリストがない状態で行うのが良いです。
私の場合はtodo-listのタスクが残っていたので、実行されませんでした。todo-listのタスクを削除すればすぐにAgent Hooksが登録されました。
:::
また.kiro/hooks/doc-test-update.kiro.hookというファイルが作成され、このような記載がされています。
```
{
  "enabled": true,
  "name": "Documentation & Test Update",
  "description": "Automatically updates documentation and runs unit tests when source files are changed",
  "version": "1",
  "when": {
    "type": "fileEdited",
    "patterns": [
      "app/**/*.tsx",
      "app/**/*.ts",
      "*.ts",
      "*.tsx",
      "package.json"
    ]
  },
  "then": {
    "type": "askAgent",
    "prompt": "A file has been modified in this Next.js project. Please:\n1. Review the changes and determine if documentation (README.md or other docs) needs to be updated\n2. If documentation updates are needed, update the relevant documentation files\n3. Run or suggest appropriate unit tests to ensure the changes work correctly\n4. Provide a summary of what was updated and any test results\n\nFocus on maintaining code quality and keeping documentation in sync with the codebase."
  }
}
```

### Agent STEERING
ステアリングは、マークダウンファイルを通じて Kiro にプロジェクトに関する永続的な知識を提供します。チャットごとに規則を説明する代わりに、ステアリングファイルを使用することで、Kiro は確立されたパターン、ライブラリ、および標準に一貫して従うようになります。
.cursor/rulesのようなものを記載する場所ですね
Generate Steering Docsを言ういかにも押してくれという主張を感じるボタンがあるので、押してみます。
そうするとこのような感じで何やら作成されたようです。
![](/images/kiro-cursor-diference/image11.png)
### product.md
製品の目的、ターゲットユーザー、主要機能、ビジネス目標を定義します。これにより、Kiroは技術的な決定の背後にある「理由」を理解し、製品目標に沿ったソリューションを提案することができます。
:::details product.md
# Product Overview

This is a Next.js web application project called "kiro-test" - a modern React-based web application built with Next.js 15 and React 19.

## Purpose
- Test/demo project for exploring Next.js capabilities
- Uses the latest Next.js App Router architecture
- Configured for modern web development with TypeScript and Tailwind CSS

## Key Features
- Server-side rendering and static generation
- Modern React 19 features
- Responsive design with Tailwind CSS
- Optimized fonts (Geist family)
- Dark mode support
- Production-ready deployment configuration
:::
### structure.md
ファイル構成、命名規則、インポートパターン、アーキテクチャ上の決定事項を概説します。これにより、生成されたコードが既存のコードベースにシームレスに適合することが保証されます。
:::details structure.md
# Project Structure

## Root Level
- `package.json` - Dependencies and scripts
- `next.config.ts` - Next.js configuration
- `tsconfig.json` - TypeScript configuration
- `postcss.config.mjs` - PostCSS/Tailwind setup

## App Directory (`app/`)
Uses Next.js App Router structure:
- `layout.tsx` - Root layout component (fonts, metadata, global structure)
- `page.tsx` - Home page component
- `globals.css` - Global styles and Tailwind imports
- `favicon.ico` - Site favicon

## Public Assets (`public/`)
Static assets served from root:
- SVG icons and logos
- Images accessible via `/filename.svg`

## Key Conventions

### File Naming
- React components: PascalCase (e.g., `RootLayout`)
- Files: kebab-case or camelCase
- TypeScript: `.tsx` for components, `.ts` for utilities

### Component Structure
- Use TypeScript interfaces for props
- Export default for page/layout components
- Metadata exports for SEO (App Router)

### Styling Approach
- Tailwind utility classes preferred
- CSS custom properties for theming
- Responsive design with mobile-first approach
- Dark mode via CSS media queries

### Import Patterns
- Next.js components: `import { Component } from "next/..."`
- Fonts: `import { Font } from "next/font/google"`
- Relative imports for local files
- Path alias `@/` available for root imports
:::
### tech.md
選択したフレームワーク、ライブラリ、開発ツール、および技術的制約を文書化します。Kiro が実装を提案する際、既存のスタックが他の選択肢よりも優先されます。
:::details tech.md
# Technology Stack

## Core Framework
- **Next.js 15.4.1** - React framework with App Router
- **React 19.1.0** - Latest React with concurrent features
- **TypeScript 5** - Type-safe JavaScript

## Styling & UI
- **Tailwind CSS 4** - Utility-first CSS framework
- **PostCSS** - CSS processing with Tailwind plugin
- **Geist Fonts** - Optimized font family (Sans & Mono)

## Development Tools
- **Turbopack** - Fast bundler for development
- **ESLint** - Code linting with Next.js config
- **TypeScript** - Strict mode enabled

## Common Commands

### Development
```bash
npm run dev          # Start development server with Turbopack
```

### Build & Deploy
```bash
npm run build        # Build for production
npm run start        # Start production server
```

### Code Quality
```bash
npm run lint         # Run ESLint checks
```

## Configuration Notes
- Uses App Router (not Pages Router)
- TypeScript strict mode enabled
- Path aliases configured (`@/*` maps to root)
- CSS custom properties for theming
- Dark mode support via CSS media queries
:::

チャットを常に日本語で返して欲しいので、こんなのも追加しました。
```steering/global.md
---
inclusion: always
---

#　プロジェクトガイドライン

## 言語設定
- **コミュニケーション**: 回答は常に日本語で行う
```
新たにルールを追加してみます
```
ディレクトリ名: 機能や役割が明確に分かる命名（例: NewFeatureListItemPage, Header）。
ファイル名: ケバブケースかつ処理や役割が分かる命名にしてください。 (button.tsx, use-theme-color.ts)
コンポーネント名: パスカルケースを使用します。（例: HeaderBreadcrumb, NewFeatureDropdown）。
```
その際の一連のやり取りを記載します。
結論：普通に公式と違ってるやんけ！
![](/images/kiro-cursor-diference/image12.png)
![](/images/kiro-cursor-diference/image13.png)
![](/images/kiro-cursor-diference/image14.png)

# MCPに関してはこちらの記事に託します
https://arc.net/l/quote/vjhlpwhz

# cursorとの比較
|  | kiro | cursor |
| ---- | ---- | ---- |
| コミットメッセージの自動生成 | ❌ | ⭕️ |
| Docs | ❌ | ⭕️ |
| Agent Hooks | ⭕️ | 🔺 （ruleに記載すれば可能かも）|
| MCP | ⭕️ | ⭕️ |
| 修正するフォルダ指定 | ❌ | ⭕️ |
| タブ保管 | ❌ | ⭕️ |
| Slack連携 | ❌ | ⭕️ |
| Git Hub連携 | ❌ | ⭕️ |

個人的にコミットメッセージの自動生成はかなり助かっているので、ぜひkiroにも導入してほしいですね〜
タブ保管は今後絶対に実装して欲しい（あったら本当にごめんなさい）
まぁまだまだプレビュー版なので、今後のアップデートに期待しましょう！

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
