---
title: "CursorのV3.2と周辺情報をまとめるんじゃ"
emoji: "🌕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cursor","AI","AI駆動開発"]
published: false
# published_at: 2026-04-30 09:30
publication_name: "genai"
---
# 初めに
分割キーボードに慣れてきたオメガマスターです。
たまにHHKBが恋しくなり、タイピングしていますが、交互でも意外に使えるようになってきました。
「お前も分割ユーザーにならないか？」（猗窩座風）
「あかざ」って入力したら「猗窩座」が第一候補に出てきた…入力したことないのに…
鬼滅パワーすごいな…

:::message
この情報は2026/5/6時点での情報です。
最新情報は公式をしっかり確認するようにしてください。
また記載順はcursorのchangelogの順番に準拠します。
:::

# モデルアクセスの制御
管理者は、モデルおよびプロバイダ単位で、より細かな許可リストやブロックリストを設定できるようになりました。プロバイダ全体をブロックすることも、速度やコンテキストウィンドウのサイズが異なる特定のモデル設定をブロックすることもできます。

企業では、新しいプロバイダやモデルバージョンをデフォルトでブロックすることもできます。
[Cursor ダッシュボードのチームのモデル設定を開いて設定](https://cursor.com/ja/dashboard/team-settings#privacy)できます。
私は管理者ではないので画面が見せられませんが、「お金のかかるモデルを使用してほしくない」という企業向けにはありがたいですね。（弊社は好きなモデルを使用していいので、あまり関係なさそうですね）

# ソフト上限とインテリジェント アラート
管理者は、ユーザーがブロックされるのを防ぐために、ハード上限ではなくソフト上限を設定できるようになりました。Cursor は使用量を監視することもでき、ソフト上限またはハード上限の 50%、80%、100% に達したユーザーに自動でアラートを送信します。
[Cursor ダッシュボードのチームのspending](https://cursor.com/ja/dashboard/team-settings#spending)から設定できるようです。
管理者ではないので、画面はありませんが、各エンジニアがしっかり使用しているか確認したい方向けにいい気がしています。
ちなみに過去のcursor meetup osakaでこんな資料がありました。一定層にはありがたそうですね〜
https://speakerdeck.com/monotaro/monotaroudecursorwodao-ru-sitemitali-xiang-toxian-shi-soretowei-lai?slide=22

# 使用量分析タブの更新
管理者は、使用量を特定のユーザーで絞り込んだり、クライアント、クラウドエージェント、オートメーション、Bugbot、Security Review などの製品環境別に内訳を表示したりできるようになりました。
[Cursorダッシュボードのusageタブ](https://cursor.com/ja/dashboard/team-settings#usage)を開いてください。
ただコード生成だけで使用しているのか、cursorを使いこなしているか判断軸の起点になりそうですね〜

# Cursor Security Review
:::message
この機能は、チーム プランおよびエンタープライズ プランでのみ利用できます。
:::
Security Reviewでは常時稼働する 2 種類のセキュリティエージェント、Security Reviewer と Vulnerability Scannerがあるようです。
- Security Review は、すべてのPRを対象に、セキュリティ上の脆弱性、認証まわりのリグレッション、プライバシーやデータ処理に関するリスク、エージェントのツールの自動承認、プロンプトインジェクション攻撃をチェックします。重大度と対処方法を添えて、該当するdiffの箇所にインラインコメントを残します。
- Vulnerability Scanner は、既知の脆弱性、古くなった依存関係、設定上の問題をチェックするために、コードベースを定期的にスキャンします。指摘の更新を Slack に送信するよう設定することもできます。

Security Review を設定するには、[Security Review ダッシュボード](https://cursor.com/ja/dashboard/security-review)を開いて、最初のエージェントを作成します。
![](/images/cursor-update-v-3_2/1.png)

またしても管理者じゃないとダメなやつ…
でもcursor automationの応用だと思うので、automationを使用している方はそんなに苦戦することなく設定できそうです。

# Cursor SDK
@cursor/sdk パッケージを使うと、自分のコードから Cursor のエージェントを呼び出せます。Cursor IDE、CLI、Web アプリで動作するのと同じエージェントを、TypeScript からスクリプトで利用できるようになりました。開発を始める際には、Cursor ネイティブの /sdk スキルも使用できます。
| 実行時 | 役割 | 使用するタイミング |
| --- | --- | --- |
| Local | Node プロセス内でエージェントをインラインで実行します。ファイルはディスクから読み込まれます。 | ワーキングツリーに対する開発用スクリプトや CI チェック。 |
| Cloud (Cursor-hosted) | 分離された VM で実行され、repo はその中にクローンされます。VM の実行は Cursor が管理します。 | 呼び出し元が repo を持っていない場合、多数のエージェントを並列で実行したい場合、または呼び出し元が切断しても実行を継続する必要がある場合。 |
| Cloud (self-hosted) | 基本的な構成は同じですが、VM は セルフホスト型プール 経由で自分で実行します。 | Cursor-hosted と同じ理由に加え、コード、シークレット、ビルドアーティファクトを自分の環境に保持する必要がある場合。 |

### 認証系
エージェントを作成する前に、CURSOR_API_KEY を設定するか、apiKey を渡してください。
SDK では、ローカル実行とクラウド実行の両方で、ユーザー API キーとサービスアカウントの API キーを使用できます。Team Admin API キーはまだサポートされていないようです。
- Cursor Dashboard → Integrations の ユーザー API キー
- チーム設定 の サービスアカウントの API キー。サービスアカウント を参照してください

### インストール
```
npm install @cursor/sdk
```
### クイックスタート
```ts
import { Agent } from "@cursor/sdk";

const agent = await Agent.create({
  apiKey: process.env.CURSOR_API_KEY!,
  model: { id: "composer-2" },
  local: { cwd: process.cwd() },
});

const run = await agent.send("Summarize what this repository does");

for await (const event of run.stream()) {
  console.log(event);
}
```

# エージェントウィンドウでマルチタスク
`/multitask` を使うと、リクエストをキューに追加する代わりに、Cursor が非同期サブエージェントを実行して並列処理します。また、大きなタスクを小さな単位に分割し、複数の非同期サブエージェントが同時に対応できるようにします。

すでにキューにメッセージがある場合も、現在の実行が終わるのを待たずに、それらに対して Cursor にマルチタスクさせることができます。
イメージはこんな感じです。
![](/images/cursor-update-v-3_2/2.png)





# 今回は割愛したもの
[チームマーケットプレイスの更新](https://cursor.com/ja/changelog#:~:text=%E6%9C%881%E6%97%A5-,%E3%83%81%E3%83%BC%E3%83%A0%E3%83%9E%E3%83%BC%E3%82%B1%E3%83%83%E3%83%88%E3%83%97%E3%83%AC%E3%82%A4%E3%82%B9%E3%81%AE%E6%9B%B4%E6%96%B0,-%E7%AE%A1%E7%90%86%E8%80%85%E3%81%AF%E3%80%81%E5%85%88%E3%81%AB)

[キャンバス](https://cursor.com/ja/changelog/page/2#:~:text=%E6%9C%8815%E6%97%A5-,%E3%82%AD%E3%83%A3%E3%83%B3%E3%83%90%E3%82%B9,-Cursor%20%E3%81%AF%E3%80%81%E5%AF%BE%E8%A9%B1)

[CLI の Debug Mode と /btw サポート](https://cursor.com/ja/changelog/page/2#:~:text=CLI%20%E3%81%AE%20Debug%20Mode%20%E3%81%A8%20/btw%20%E3%82%B5%E3%83%9D%E3%83%BC%E3%83%88)
