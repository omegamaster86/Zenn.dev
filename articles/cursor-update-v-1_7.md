---
title: "CursorのV1.7とBrowser Automationについて調査したんじゃ"
emoji: "🌕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cursor","AI","AI駆動開発"]
published: false
# published_at: 2025-09-22 09:30
---
# 初めに
cursorのV1.7がリリースされました！
個人的にBrowser Automationの方に興味がありますが、アップデート情報もしっかりと見ていきましょう〜
:::message
pro人プランからTeamプランに変更したため、実機でテストできていないことがありますが、ご容赦ください。
またこの情報は2025/10/05時点の情報なので、参考までにご覧ください。
:::

# 1.7で発表された情報
### Agentオートコンプリート
チャット入力欄でTabを押すと提案が表示され、最近開いたファイルやメソッド、変数などを容易にコンテキストへ添付できるようになる。
まぁコードを書くときのtab機能と同じものが、チャットでも利用できるようになったと思って良いです。
![](/images/cursor-update-v-1_7/1.png)

## Hooksによるエージェントループの制御（ベータ版）
Hooksを使うと、カスタムスクリプトでエージェントループを監視・制御・拡張できます。Hooksは起動されたプロセスで、双方向に JSON を用いて stdio 経由で通信します。エージェントループの定義済みステージの前後で実行され、挙動の観察、ブロック、変更が可能です。
公式が意見を募集しているので、是非是非みなさんの意見を届けましょう。
![](/images/cursor-update-v-1_7/2.png)
Hooksを使用すると下記のことが可能になります。
- 編集後にフォーマッターを実行
- イベントにアナリティクスを追加
- PII やシークレットをスキャン
- リスクの高い操作をゲート（例: SQL 書き込み）

### クイックスタート
`~/.cursor/hooks.json`に`hooks.json`ファイルを作成する。
```
{
  "version": 1,
  "hooks": {
    "afterFileEdit": [
      { "command": "./hooks/format.sh" }
    ]
  }
}
```
`~/.cursor/hooks/format.sh`にフック用スクリプトを作成します。
```
#!/bin/bash
# 入力を読み取り、何かを実行し、0で終了
cat > /dev/null
exit 0
```
下記を実行することで可能にする。
`chmod +x ~/.cursor/hooks/format.sh`
再起動後、ファイルを編集するたびにフックが実行されます。
### 設定
`hooks.json`ファイルでフックを定義します。設定は複数レベルで指定でき、優先度の高いソースが低いソースを上書きします。
```
~/.cursor/
├── hooks.json
└── hooks/
    ├── audit.sh
    ├── block-git.sh
    └── redact-secrets.sh
```
ホームディレクトリ（ユーザー管理）:`~/.cursor/hooks.json`
`hooks`オブジェクトは、フック名をフック定義の配列に対応付けます。各定義は現在 `command`プロパティをサポートしており、シェル文字列、絶対パス、または`hooks.jsonファイル`からの相対パスを指定できます。
```
{
  "version": 1,
  "hooks": {
    "beforeShellExecution": [
      { "command": "./script.sh" }
    ],
    "afterFileEdit": [
      { "command": "./format.sh" }
    ]
  }
}
```
公式に複数のhookイベントが記載されていますので、是非見に行ってみてください
https://cursor.com/ja/docs/agent/hooks#
これが個人的に役に立ちそうな気がしています。
beforeShellExecution / beforeMCPExecution
任意のシェルコマンドまたはMCPツールの実行前に呼び出されます。許可の可否を返します。
```
// beforeShellExecution 入力
{
  "command": "<完全なターミナルコマンド>",
  "cwd": "<現在の作業ディレクトリ>"
}

// beforeMCPExecution 入力
{
  "tool_name": "<ツール名>",
  "tool_input": "<JSONパラメータ>"
}
// 加えて以下のいずれか:
{ "url": "<サーバーURL>" }
// または:
{ "command": "<コマンド文字列>" }

// 出力
{
  "permission": "allow" | "deny" | "ask",
  "userMessage": "<クライアントに表示されるメッセージ>",
  "agentMessage": "<エージェントに送信されるメッセージ>"
}
```

## チームルールとルール管理
チームはダッシュボードから、すべてのプロジェクトに適用されるグローバルルールを定義・共有できるようになりました。さらに、Bugbot向けのチームルールも提供し、リポジトリ間で一貫した挙動を実現します。
管理者権限で設定できるらしいので、公式から画像を引っ張ってきます。ruleの共有がなくなるのは嬉しいですね〜
![](/images/cursor-update-v-1_7/3.png)
個人的にはTeamプランのユーザーのトークン使用量をアップしてほしいですね〜proプランと同じ20$相当の使用量なので、ユーザー的にはTeamプランの旨みがない…
会社的にメリットはあると思いますが、ユーザーへのメリットがあっても良いと思いますけどね〜
