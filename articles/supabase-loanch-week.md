---
title: "SupabaseのLaunch Weekがあったんじゃ！"
emoji: "💰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["React","Next.js","TypeScript","Supabase",]
published: true
published_at: 2025-04-13 16:30
---

# 初めに
私が勤めている会社ではSupabaseを積極的に採用しているので、今回は2025年3月31日～4月4日に行われたSupabaseのLaunch Weekの内容を見ていこうかなと思いまとめてみました〜
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
### エッジ関数のダウンロード
`supabase functions download FUNCTION_NAME`Supabase CLI を使用するか、ダッシュボードのダウンロードボタンをクリックすると、Edge Functionsのソース コードをダウンロードできます。
![](/images/supabase-loanch-week/image11.png)
:::message
ダッシュボードのEdgeFunctionsエディタは現在、バージョン管理やロールバックをサポートしていません。簡単なテストやプロトタイプ作成にのみ使用することをお勧めします。本番環境への移行準備ができたら、EdgeFunctionsのコードをソースコードリポジトリ（例：git）に保存し、CI統合のいずれかを使用してデプロイしてください。
:::
### SupabaseダッシュボードからのEdge関数のテスト
SupabaseダッシュボードからEdge Functionsをテストするための組み込みツールを導入しました。さまざまなリクエストペイロード、ヘッダー、クエリパラメータを使用してEdgeFunctionsを実行できます。組み込みテスターは、レスポンスステータス、ヘッダー、本文を返します。
![](/images/supabase-loanch-week/image12.png)
組み込みのエディターとテスターを使用すると、Supabase ダッシュボードを離れることなく、Edge関数を作成、テスト、リファクタリングするための合理化されたワークフローを実現できます。
### EdgeFunctionsのデプロイにDockerは不要になりました
多くのご要望にお応えして、Supabase CLI から`--use-api`フラグを使用してEdgeFunctionsをデプロイできるようになりました。この設定ではDockerは使用されません。今後のリリースでは、これをデフォルトの動作にする予定です（`--use-docker`フォールバックオプションとして フラグを指定可能）。
```
supabase functions deploy MY_FUNCTION --use-api
```
### エッジ機能をデプロイするための新しい API
EdgeFunctionsエディタとSupabase CLIの両方でDockerを使わずにデプロイできるようになったのは、Edge Functionsのデプロイとアップデート用に導入された新しいAPIのおかげです。これらのAPIは一般公開されており、カスタム統合やワークフローの構築にご利用いただけます。
これらの API エンドポイントの詳細と公式リファレンスについては、[Changelogのアナウンス](https://github.com/orgs/supabase/discussions/33720)を確認してください。
### Deno 2.1 プレビュー
最後に、Supabase Edge RuntimeにDeno 2.1のサポートを追加しました。Deno2.1では、組み込みのDenoコマンドを使用して、新規プロジェクトのスキャフォールディング、依存関係の管理、テストの実行、Lint を実行できます。
Edge 関数で[Deno2.1 ツールを使い始める方法については、ガイド](https://github.com/orgs/supabase/discussions/34054)をご覧ください。

# 3日目：[リアルタイム: データベースからのブロードキャスト](https://supabase.com/blog/realtime-broadcast-from-database)
これで、リアルタイム ブロードキャストを使用して、[データベースからのブロードキャスト](https://supabase.com/docs/guides/realtime/broadcast#broadcast-from-the-database)でクライアントに送信されたデータベースの変更をスケーリングできるようになりました。
### Supabase Realtime とは何ですか?
Supabase Realtime を使用すると、通知、チャット、ライブ カーソル、共有ホワイトボード、マルチプレイヤーゲームなどの没入型機能を構築したり、データベースの変更を聴いたりすることができます。
Realtimeには次の機能が含まれます。
- ブロードキャスト、クライアントライブラリ、REST、またはデータベースを使用して低遅延メッセージを送信します。
- プレゼンス：オンラインユーザーの状態をクライアント間で一貫して保存および同期します。
- Postgresの変更は、データベースをポーリングし、変更をリッスンし、クライアントにメッセージを送信します。
データベースからのブロードキャストは最新の改善点です。Postgresの変更よりも初期設定は必要ですが、より多くのメリットがあります。
- 特定のアクションをターゲットにすることができます`（INSERT、、、）``UPDATE``DELETE``TRUNCATE`
- 完全な記録の代わりに、メッセージの本文で送信する列を選択します
- SQLを使用して特定のチャネルにデータを選択的に送信します
データベースの変更を使用してリアルタイム アプリケーションを構築するには、次の2つのオプションがあります。
- データベースからのブロードキャストは、データベース自体の変更によってトリガーされるメッセージを送信します。
- Postgresの変更、データベースの変更をポーリングする
![](/images/supabase-loanch-week/image13.png)
### データベースからのブロードキャスト
次のようなシナリオでは、Postgresの変更の代わりにデータベースからのブロードキャストを使用することをお勧めします。
- 多くの接続ユーザーがいるアプリケーション
- 完全な記録を提供する代わりに、メッセージのペイロードをサニタイズする
- 送信メッセージの遅延の削減
データベースからのブロードキャストを設定する方法を見ていきましょう。
まず、行レベル セキュリティ (RLS) ポリシーを設定して、関連するメッセージへのユーザー アクセスを制御します。
```
create policy "Authenticated users can receive broadcasts"
on "realtime"."messages"
for select
to authenticated
using ( true );
```
次に、データベースの変更が検出されるたびに呼び出される関数を設定します。
```
create or replace function public.your_table_changes()
returns trigger
as $$
begin
    perform realtime.broadcast_changes(
	    'topic:' || new.id::text,   -- topic
		   tg_op,                          -- event
		   tg_op,                          -- operation
		   tg_table_name,                  -- table
		   tg_table_schema,                -- schema
		   new,                            -- new record
		   old                             -- old record
		);
    return null;
end;
$$ language plpgsql;
```
次に、関数を実行するトリガー条件を設定します。
```
create trigger broadcast_changes_for_your_table_trigger
after insert or update or delete
on public.your_table
for each row
execute function your_table_changes();
```
最後に、変更をリッスンするようにクライアント コードを設定します。
```
const id = 'id'
await supabase.realtime.setAuth() // Needed for Realtime Authorization
const changes = supabase
  .channel(`topic:${id}`, {
    config: { private: true },
  })
  .on('broadcast', { event: 'INSERT' }, (payload) => console.log(payload))
  .on('broadcast', { event: 'UPDATE' }, (payload) => console.log(payload))
  .on('broadcast', { event: 'DELETE' }, (payload) => console.log(payload))
  .subscribe()
```
詳しい情報と使用例については、必ず[ドキュメント](https://supabase.com/docs/guides/realtime/broadcast)をお読みください。
### データベースからのブロードキャストはどのように機能しますか?
データベースからのリアルタイムブロードキャストは、テーブルに作成されたパブリケーションに対してレプリケーションスロットを設定します`realtime.messages。`これにより、リアルタイムは新しい行が挿入されるたびに、先行書き込みログ（WAL）の変更をリッスンできるようになります。
Realtime は WAL 内に新しい挿入を検出すると、そのメッセージをすぐにターゲット チャネルにブロードキャストします。
2 つのヘルパー関数を作成しました。
- realtime.sendrealtime.messages:テーブルにメッセージを追加するシンプルな関数
- realtime.broadcast_changes: Postgresの変更に似たペイロードを作成するより高度な関数
この`realtime.send関数`はトリガー内で安全に動作するように設計されています。例外をキャッチし、`pg_notify`エラー情報をRealtimeサーバーに送信して適切なログ出力を実現します。これにより、何か問題が発生してもトリガーが動作しなくなるのを防ぎます。
![](/images/supabase-loanch-week/image14.png)
これらの改善により、データベースの変更を数万の接続ユーザーに一括でサブスクライブすることが可能になりました。また、以下のような新たな用途も可能になりました。
- データベース関数から直接ブロードキャストする
- 接続されたクライアントに特定のフィールドのみを送信する
- [Supabase Cron](https://arc.net/l/quote/pamypvdu)を使用してスケジュールされたイベントを作成する
これらすべてにより、リアルタイム アプリケーションはより高速かつ柔軟になります。
# 4日目：[よりシンプルなデータベース管理のための宣言型スキーマ](https://supabase.com/blog/declarative-schemas)
複雑なデータベーススキーマの管理と保守を簡素化する宣言型スキーマをリリースします。宣言型スキーマを使用すると、データベース構造を明確かつ一元管理されたバージョン管理された方法で定義できます。
### 宣言型スキーマとは何ですか?
宣言型スキーマは、データベースの最終的な望ましい状態を、`.sql`プロジェクトと一緒に保存およびバージョン管理できるファイルに保存します。例えば、クラシック`products`テーブルの宣言型スキーマは次のとおりです。
```
create table "products" (
  id         bigint generated by default as identity,
  name       text not null,
  category   text,
  price      numeric(10, 2) not null,
  created_at timestamptz default now()
);

alter table "products"
enable row level security;
```
宣言型スキーマを使用すると、データベース スキーマを直接変更する場合に比べて、次のような多くの利点があります。
- 単一の管理画面。データベース スキーマ全体を 1 か所で管理することで、冗長性と潜在的なエラーを削減します。
- バージョン管理された移行。移行ファイルを自動的に生成し、環境間で更新されたスキーマの一貫性を確保します。宣言型スキーマファイルは、プロジェクトファイルと共にバージョン管理システムに保存します。
- 簡潔なコードレビュー。複雑な移行スクリプトを手動で繰り返すことなく、テーブル、ビュー、関数への変更を簡単に確認できます。
### 宣言型スキーマと移行
[データベースへの変更を追跡・適用するには、移行機能](https://supabase.com/docs/guides/deployment/database-migrations)を使用するのがベストプラクティスです。変更を加えるたびに、すべての変更内容を含む新しいファイルが作成され、変更のバージョン管理と再現性が維持されます。
ただし、データベース スキーマの複雑さが増すにつれて、データベース スキーマ全体を確認できる場所がなくなるため、バージョン管理された移行を使用して開発することがますます困難になります。
例えば、Supabaseには複雑で頻繁に更新されるprojectsテーブルがあります。RLSを有効にすると、以下のように表示されます。
```
create table private.projects (
  id              bigint    not null,
  name            text      not null,
  organization_id bigint    not null,
  inserted_at     timestamp not null,
  updated_at      timestamp not null
);

alter table private.projects
enable row level security;

create policy projects_insert
  on private.projects
  for insert
  to authenticated
with check auth.can_write(project_id);

create policy projects_select
  on private.projects
  for select
  to authenticated
using auth.can_read(project_id);

-- Users can only view the projects that they have access to
create view public.projects as select
  projects.id,
  projects.name,
  projects.organization_id,
  projects.inserted_at,
  projects.updated_at
from private.projects
where auth.can_read(projects.id);
```
テーブルprojectsはプライベートスキーマに作成され、読み取り用に公開されたパブリックビューが提供されます。RLSポリシー上に属性ベースのアクセス制御（ABAC）が実装されているため、クエリはユーザーがアクセス権を持つプロジェクトのみを返します。
Postgresのビューはデフォルトでは更新できないため、Supabaseユーザーが新しいプロジェクトを作成したときに、基になるテーブルへの書き込みをカスケードするトリガー関数を定義しました。これにより、基になるprojectsテーブルに対して適切なRLSポリシーを適用しながら、通常のPostgREST呼び出しでビューを挿入できるため、開発が容易になります。
```
-- Triggers to update views from PostgREST: select('projects').insert({ ... })
create function public.public_projects_on_insert() returns trigger
as $$
begin
  insert into private.projects(
    name,
    organization_id,
    inserted_at,
    updated_at
  ) values (
    NEW.name,
    NEW.organization_id,
    coalesce(NEW.inserted_at, now()),
    coalesce(NEW.updated_at, now())
  ) returning * into NEW;
  return NEW;
end
$$ language plpgsql;

create trigger public_projects_on_insert
  instead of insert
  on public.projects
  for each row
execute function public.public_projects_on_insert();
```
この複雑さは開発速度を低下させます。テーブルへの変更によって他のビューや関数が動作しなくなる可能性があるためです。2022年初頭には、新しい列を追加するという単純な変更に、以下の手順が必要でした。
1. projects移行ファイルまたはデータベースを照会して、テーブルの最新のスキーマを見つけます。
2. alter table新しい移行ファイルにステートメントを書き込みます。
3. ビュー定義をコピーして更新しprojects、新しい列を含めます。
4. トリガー関数の定義をコピーして更新し、新しい列を含めます。
5. 新しい pgTAP テストを追加し、既存のテストが合格することを確認します。
6. 少なくとも数百行になる新しい移行ファイルをレビュー用に送信します。
このプロセスは面倒で、複数のエンジニアが同時に作業するのはストレスがたまる作業ですprojects。プルリクエストをマージするとマージ競合が発生し、手順1～5を繰り返して解決する必要があります。
### 本番環境での宣言型スキーマの使用
宣言型スキーマの採用により、エンジニアはデータベーススキーマの更新を単一の画面で管理できるようになりました。移行ファイル内で影響を受けるPostgreSQLエンティティを手動で複製する代わりに、スキーマ定義を1か所で変更するだけで済みます。
次に、 migraなどのスキーマ比較ツールを使用して、移行ファイルを生成するときにビューと関数に必要な更新を判断します。
たとえば、テーブル`metadata`に新しい列を追加すると`projects`、1 行の差分になります。
```
--- a/supabase/schemas/projects.sql
+++ b/supabase/schemas/projects.sql
@@ -2,6 +2,7 @@ create table private.projects (
   id              bigint    not null,
   name            text      not null,
   organization_id bigint    not null,
+  metadata        jsonb,
   inserted_at     timestamp not null,
   updated_at      timestamp not null
 );
```
同じプロセスは、ビュー、データベース関数、RLSポリシー、ロール付与、カスタム型、制約にも適用されます。生成された移行ファイルには依然として手動レビューが必要ですが、開発時間は数時間から数分に短縮されました。また、他のプルリクエストによって発生したマージ競合のリベースも大幅に容易になりました。
### 宣言型スキーマを使い始める
宣言型スキーマは現在 Supabase で利用可能です。
データベース スキーマを 1 か所で管理し、バージョン管理された移行を生成する方法については、[ドキュメント](https://supabase.com/docs/guides/local-development/declarative-database-schemas)をお読みください。
宣言型スキーマを使用して開発する場合のより優れたツールとIDE統合については、[Postgres Language Server](https://supabase.com/blog/postgres-language-server)に関するブログ投稿をご覧ください。
# 5日目：[Supabase MCP サーバー](https://supabase.com/blog/mcp-server)
[Supabase公式](https://github.com/supabase-community/supabase-mcp)MCPサーバーをリリースしました。このサーバーを使えば、お気に入りのAIツール（ CursorやClaudeなど）をSupabaseに直接接続できます。
### MCP サーバーとは何ですか?
[MCPはModel Context Protocol](https://modelcontextprotocol.io/)の略称です。大規模言語モデル（LLM）がSupabaseのようなプラットフォームと通信する方法を標準化します。
当社の MCP サーバーは、AI ツールを Supabase に接続し、データベースの起動、テーブルの管理、構成の取得、データのクエリなどのタスクを AI ツールが代わりに実行できるようにします。
たとえば、次の Cursor は Next.js + Supabase アプリを構築し、Supabase URL と匿名キーを取得し、.env.localNext.js が使用できるようにファイルに保存します。
MCPサーバーでは、いわば「アビリティ」のようなツールを使用します。Supabase MCPサーバーでは20種類以上のツールが利用可能です。
- テーブルを設計し、移行を使用して追跡する
- SQLクエリを使用してデータを取得し、レポートを実行する
- 開発用のデータベース ブランチを作成する (実験的)
- プロジェクト構成を取得する
- Supabaseの新しいプロジェクトを立ち上げる
- プロジェクトの一時停止と復元
- 問題をデバッグするためにログを取得する
- データベーススキーマに基づいて TypeScript 型を生成する
機能の完全なリストについては、プロジェクトの README の[ツールを参照してください。](https://github.com/supabase-community/supabase-mcp#tools)
### 設定
次の JSON を使用して、ほとんどの AI クライアントで Supabase MCPを設定できます。
```:cursor/mcp.json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": [
        "-y",
        "@supabase/mcp-server-supabase@latest",
        "--access-token",
        "<personal-access-token>"
      ]
    }
  }
}
```
このフィールドに[個人アクセストークン（PAT）](https://supabase.com/dashboard/account/tokens)を作成する必要があります<personal-access-token>。このトークンは、Supabaseアカウントを使用してMCPサーバーを認証します。
:::message
一部のクライアントでは若干変更されたJSON形式が想定されるため、Windowsユーザーはこのコマンドの前に を付ける必要がありますcmd /c。各クライアントとOSごとの詳細な手順については、[MCPドキュメント](https://supabase.com/docs/guides/getting-started/mcp)をご覧ください。
:::
### MCP はどのように機能しますか?
MCP を初めて使用する場合は、このプロトコルを詳しく調べて、それがどのようにして生まれたのか、どのような機能を提供しているのかを理解してみる価値があります。
今日の大規模言語モデル（LLM）のほとんどは「ツール呼び出し」をサポートしています。これは、会話のコンテキスト（「ヒューストンの天気はどうですか？」など）に基づいて、モデルが開発者が提供するツール（などget_weather）を呼び出すことを選択する機能です。これにより、LLMがユーザーに代わって外界と対話するツールを呼び出す、エージェントのようなエクスペリエンスへの道が開かれました。
開発者は、ツールとそれらが受け入れるパラメータを含む JSON スキーマを提供することで、LLM に利用可能なツールを伝えます。
```
{
  "name": "get_weather",
  "description": "Get the current weather for a specific location",
  "parameters": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "The city and state/country, e.g. 'San Francisco, CA' or 'Paris, France'"
      }
    },
    "required": ["location"]
  }
}
```
このJSONスキーマ形式は、ほとんどのLLMで標準となっています。しかし、重要なのは、各ツールの実装が標準ではないということです。get_weatherリクエストを満たすために、ツール呼び出しをどこかの天気予報APIに接続するかどうかは、開発者の責任となります。
そのため、開発者はプロジェクトやアプリ間でツールの実装を重複させてしまうことがよくありました。また、どのアプリでも、エンドユーザーは開発者が厳選したツールしか利用できませんでした。ユーザーが独自のツールを持ち込めるプラグイン型のツールエコシステムを構築する機会はありませんでした。
### MCP標準
MCPはツールエコシステムを標準化することでこの問題を解決します。つまり、クライアント（例：Cursor）とツールプロバイダー（例：Supabase）の両方が理解できるプロトコルを作成し、両者を分離します。CursorはMCP仕様のクライアント側を実装するだけで、そのLLMはMCPを実装したあらゆるサーバーと即座に連携します。Cursor（そして他のAIアプリ）は、ユーザーがMCPを実装している限り、独自のツールを持ち込むことができます。
### リソースとプロンプト
MCPには、ツール呼び出し以外にも、リソースとプロンプトといった（オプションの）プリミティブが組み込まれています。リソースを利用することで、サーバーはクライアントが読み取り、LLMのコンテキストとして使用できる任意のデータやコンテンツを公開できます。具体的には、以下のようなものが挙げられます。
- ファイルの内容
- データベースレコード
- APIレスポンス
- ライブシステムデータ
- スクリーンショットと画像
- ログファイル
- その他
プロンプトを使用すると、サーバーは再利用可能なプロンプトテンプレートを定義し、クライアントがユーザーやLLMに表示できるようになります。これにより、サーバーはLLMがサービスとやり取りする際に、カスタム指示やベストプラクティスを定義できるようになります。
現在、ほとんどのクライアントはツールプリミティブのみをサポートしています。アプリがリソースやプロンプトをどのように採用していくのか、そしてそれらの環境におけるSupabase MCPエクスペリエンスをさらに向上させていくのか、楽しみにしています。
モデル コンテキスト プロトコルの詳細については、[公式の MCPドキュメント](https://modelcontextprotocol.io/introduction)を参照してください。
### 次は何?
MCPはAIビルダーの世界において大きな可能性を秘めていると信じており、今後も投資を継続していきたいと考えています。ロードマップに予定されている機能の一部をご紹介します。
### エッジ関数の作成とデプロイ
Supabase Edge Functionsを使用すると、ユーザーに最も近いエッジネットワークから、カスタムサーバーサイドコードを実行できます。Supabaseダッシュボードから直接Edge Functionsを作成・デプロイできる機能を発表しました。お気に入りのAIアシスタントから直接Edge Functionsを作成・デプロイするのも当然のことでしょう。
### ネイティブ認証
MCP仕様の最新版（執筆時点では2025年3月26日）には、公式の[認証サポート](https://spec.modelcontextprotocol.io/specification/2025-03-26/basic/authorization/)が含まれています。これは、現在サーバー用の個人アクセストークンを手動で作成する必要があるのとは異なり、将来のバージョンでは標準のOAuth 2ログインフローを使用してSupabaseで認証できるようになることを意味します。このフローは次のようになります。
1. Supabase MCP を AI アプリ/IDEに接続します
2. AIアプリがブラウザウィンドウを開き、Supabaseに直接ログインします。
3. ログインに成功すると、AIアプリに戻り、完全に認証されます。
これは、現在アプリで見られる他の「Xでログイン」OAuthフローと何ら変わりません。これにより、MCPのセットアップが簡素化されるだけでなく、Supabaseアカウントへのよりきめ細やかなアクセスも可能になると考えています。
### より優れたスキーマ検出
AIを使用してデータベースを設計する際には、LLMに既存のスキーマへのアクセスを許可することで、変更（ など`alter table`）を行う際に実行すべきSQLを正確に把握できるようになります。現在、LLMがテーブルをフェッチするために使用できるツールは1つしか提供されていません`list_tables`。これは便利ですが、ビュー、トリガー、関数、ポリシーなど、他にも多くのデータベースオブジェクトが用意されており、それらもすぐに利用できるようにしておく必要があります。
現在、LLM がこれらの他のオブジェクトのいずれかにアクセスする必要がある場合、汎用`execute_sql`ツールを実行して からそれらを取得できます`information_schema`。たとえば、データベース内のすべてのトリガーを取得するには、LLM は次のように実行します。
```
select
    event_object_schema as schema,
    event_object_table as table,
    trigger_name,
    event_manipulation as event,
    action_statement as definition
from
	information_schema.triggers
order by
	event_object_schema, event_object_table;
```
この方法は多くの場合うまく機能しますが、LLMが`information_schema`テーブルの正確な構造を把握する必要があり、冗長なSQLクエリのために多くのトークンを消費し、結果の解析時にエラーが発生する可能性があります。より構造化された簡潔なクエリ方法を採用することで、検出可能性が向上し、過剰なトークンの使用を削減できると考えています。今後の展開にご期待ください。
### さらなる保護
データベース開発においてAIを過信しすぎる人もいるかもしれません。Supabaseは[データベースブランチ](https://supabase.com/docs/guides/deployment/branching)をサポートしており、新機能の設計時に別々の開発データベースを立ち上げ、準備が整ったらそれらをマージすることができます。つまり、何か大きな問題が発生した場合でも、ブランチを以前のバージョンに簡単にリセットし、中断したところから再開できます。
当社の MCP サーバーは現在すでにブランチをサポートしていますが、破壊的な操作を自動検出し、実行前に確認を要求するなど、さらに多くの保護機能を追加できると考えています。
### Supabase MCP を使い始める
私たちは、AI ツールとデータベース間のギャップを埋め、ツール間のコンテキスト切り替えではなく構築に集中できるようにするために Supabase MCP サーバーを構築しました。
MCPプロトコルは、長時間接続を必要とせずに完全にステートレスなサーバーをサポートする新しい[Streamable HTTPトランスポートなどの提案によって進化しています。私たちはこれらの開発を注意深く追跡し、Supabase MCPエクスペリエンスにどのようなメリットをもたらすかを評価しています。](https://github.com/modelcontextprotocol/specification/pull/206)
問題が発生した場合や、追加すべき新しいツールのアイデアをお持ちの場合は、 GitHubリポジトリで[Issue](https://github.com/supabase-community/supabase-mcp/issues/new)を開いてください。特に、スキーマ検出、データベースブランチ、その他の安全機能に関するご意見をお待ちしております。これらの機能の保護機能の改良に引き続き取り組んでまいります。
Supabase MCP で構築できるものの最新のアップデートと例については、[ドキュメント](https://supabase.com/docs/guides/getting-started/mcp)をご覧ください。
